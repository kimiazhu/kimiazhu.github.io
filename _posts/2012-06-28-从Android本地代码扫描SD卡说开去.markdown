---
layout:     post
title:      "从Android本地代码扫描SD卡说开去"
date:       2012-06-28 20:18
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - JNI
    - NDK
    - C
    - C++
    - Java
---

# 一、缘起
图丫丫需要一个扫描sd卡的功能，其实系统本身有这样的功能，经过评估之后决定建立自己的一套，原因主要有三：

1. 系统媒体库各个应用都能修改，其中信息错综复杂，和实际sd卡上的媒体文件比较，或多或少，或错或乱，有不少是问题数据。
2. 在Android4.0的部分机器上有缺陷，这个缺陷是系统引起的。原本DCIM目录下会有这么一个文件：**/mnt/sdcard/DCIM/.thumbnails/.thumbdata3--1967290299**。当这个缺陷被激活的时候，系统生成该文件的逻辑会产生错乱，几乎是陷入死循环，网上有不少朋友反馈该文件有几百M，甚至1、2GB，并且在生成过程中可能导致系统无响应，甚至系统自动重启。触发该缺陷的操作之一，是生成缩略图，或者从媒体库获取缩略图。
3. 希望能够独立并且更灵活控制系统图片信息。例如对SD数据的完整重建和分析。

# 二、实施
扫描是一件相当繁重的任务，当然我们可能尽量避免一些不必要的目录，例如tmp目录，点开头的隐藏目录，还有目录底下有.nomedia文件的，都可以忽略。这样多少可以加速扫描过程。这里对实现过程中的一些性能问题做下记录分析。

## 1. Java VS C
效率最大的问题就是Java代码和本地C代码的区别。应用启动首次扫描SD卡，内部共有14000多个文件（包括目录），分别使用Java和C进行遍历。

#### A) Java遍历
最普通的Java代码，仅仅是列举并计数，不做任何其他处理，外层起线程并即时。

```java
/** Called when the activity is first created. */
@Override
public void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	new Thread(new Runnable() {
        @Override
        public void run() {
            long _start = System.currentTimeMillis();
            listFile(new File("/sdcard"));
            long elapse = System.currentTimeMillis() - _start;
            Log.w("ELAPSE", String.format("in Java cost: %s, total file count: %s", 
                    elapse, fileCount));
        }
    }).start();
}

/**
 * 遍历文件
 * 
 * @param file 根目录
 */
private void listFile(final File file) {
    File[] files = file.listFiles();
    if (files != null && files.length > 0) {
        for (File f : files) {
            if (f.isDirectory()) {
                listFile(f);
            } else {
                fileCount++;
            }
        }
    }
}
```

#### B) C遍历
NDK的代码相对繁琐，我们仅以最基本的功能进行演示和对比，实际产品化的代码要处理更多的事情。

```c
/* 
 * 列举文件并计数
 */
void listDir(JNIEnv * env, jobject cb, sqlite3 * db, char * path)
{
	DIR              *pDir ;
	struct dirent    *ent  ;
	char              childpath[2048];
	struct stat       stat_buf;
	
	pDir=opendir(path);
	memset(childpath,0,sizeof(childpath));
	
	while((ent=readdir(pDir))!=NULL)
	{
		if(ent->d_type & DT_DIR)
		{
			if(!isValidPath(ent->d_name))
				continue;
			sprintf(childpath,"%s/%s",path,ent->d_name);
	
			if(stat(childpath, &stat_buf) == -1)
				continue;
			if((stat_buf.st_mode & 0777) <= 04)
				continue;
			listDir(env, cb, db, childpath);
		}
		count++;
	}
	closedir(pDir);
	pDir = NULL;
}
```

####C) 结论

1. 06-28 21:14:17.421: W/ELAPSE(6198): in Java cost: 21613, total file count: 14253
2. 06-28 21:30:40.210: W/ELAPSE(6921): in C cost: 8475, total file count: 14253
3. 本地代码做同样的事情，耗时是Java的1/2至1/3之间。当然，Java本身列举文件的实现可能还有更多的判断以便更通用和健壮，但是就列举目录这个操作而言，保守估计本地代码至少比Java代码执行快1倍。

## 2. 数据库写入：单条 VS 批量
前面的代码除了做列举，没有做过滤，也没有任何其他的操作。实际应用中我们需要将图片写到数据库，以便被其他应用调用。

两种方式写入，一种是找到一个文件，写入一次，一种是打开一个事务，写入所有的记录，然后再提交并关闭数据库。

#### A)逐条写入
逐条写入的最大时间损耗在数据库连接方面，每次打开和关闭数据库相当大，这种情况下效率较低。

```c

#define CALL_SQLITE(env,cb,f)                               \
{                                                           \
    int i;                                                  \
    i = sqlite3_ ## f;                                      \
    if (i != SQLITE_OK) {                                   \
    	char log[512];						                \
    	const char *msg = sqlite3_errmsg (db);              \
        sprintf (log, "%s failed with status %d: %s\n",     \
                 #f, i, msg);              	                \
        if(DEBUG)LOGI(log);					                \
    }                                                       \
}                                                           \

int insertToDatabase(char * path, long timestamp)
{
	sqlite3 * db;
	char * sql;
	sqlite3_stmt * stmt;
	int nrecs;
	char * errmsg;
	int i;
	
    CALL_SQLITE (open ("/sdcard/database.db", & db));
	sql = "INSERT INTO images (path, modified) VALUES (? , ?)";
	sqlite3_prepare_v2 (db, sql, strlen (sql) + 1, & stmt, NULL);
	sqlite3_bind_text (stmt, 1, path, strlen(path) + 1, SQLITE_STATIC);
	char time[sizeof(int)];
	sprintf(time, "%i", timestamp);
	sqlite3_(bind_text (stmt, 2, time, strlen(time) + 1, SQLITE_STATIC);
	sqlite3_(step (stmt));
	
    if(DEBUG)LOGI ("row id was %d\n", (int) sqlite3_last_insert_rowid (db);
	sqlite3_finalize(stmt);
    sqlite3_close (db);
	return 0;
}
```

#### B)批量操作
批量操作旨在减少数据库的打开和关闭的开销，以事务的方式做，一次性打开事务，然后循环插入记录，最后进行提交，如果中间有失败，则忽略之。

```c

#define CALL_SQLITE(env,cb,f)                               \
{                                                           \
    int i;                                                  \
    i = sqlite3_ ## f;                                      \
    if (i != SQLITE_OK) {                                   \
    	char log[512];						                \
    	const char *msg = sqlite3_errmsg (db);              \
        sprintf (log, "%s failed with status %d: %s\n",     \
                 #f, i, msg);              	                \
        if(DEBUG)LOGI(log);					                \
    }                                                       \
} 
    
int insertToDatabase(JNIEnv * env, jobject cb, sqlite3 * db, char * path, long timestamp)
{
	char * sql;
	sqlite3_stmt * stmt;
	int nrecs;
	char * errmsg;
	int i;
	
	sql = "INSERT INTO images (path, modified) VALUES (? , ?)";
	CALL_SQLITE (env, cb, prepare_v2 (db, sql, strlen (sql) + 1, & stmt, NULL));
	CALL_SQLITE (env, cb, bind_text (stmt, 1, path, strlen(path) + 1, SQLITE_STATIC));
	char time[sizeof(int)];
	sprintf(time, "%i", timestamp);
	CALL_SQLITE (env, cb, bind_text (stmt, 2, time, strlen(time) + 1, SQLITE_STATIC));
	CALL_SQLITE_EXPECT (env, cb, step (stmt), DONE);
	
	CALL_SQLITE (env, cb, finalize(stmt));
	return 0;
}
	
void listDir(JNIEnv * env, jobject cb, sqlite3 * db, char * path)
{
	//见之前的代码，核心逻辑相同。
}
	
Java_com_example_MediaScanner_scanImages(JNIEnv * env, jobject thiz)
{
	sqlite3 * db;
	CALL_SQLITE (env, callback, open ("/sdcard/database.db", & db));
	CALL_SQLITE(env, callback, exec(db, "BEGIN", 0, 0, 0));
	listDir(env, callback, db, path);
	CALL_SQLITE(env, callback, exec(db, "COMMIT", 0, 0, 0));
	CALL_SQLITE (env, callback, close (db));
}
```

#### C) 结论

1. 逐条操作的耗时大概在100ms每条，
2. 批量操作的耗时可以下降到10ms以下。

# 三、总结
当然，实际的应用中有更多要考虑的问题，其中包括文件和目录的过滤、更详细的权限判断、部分图片信息的读取以及写入到数据库，这些都是耗时操作。另外就是线程同步的问题，开启事务的情况下，事务未提交时必须阻止其他线程的竞争。另外数据扫描和插入数据库是否正常完成，以及出错的情况也应该通过回调方法通知前端的Java程序。另外数据库之中也少不了建一些约束，等等等等，这些操作在实际生产代码时都会影响到效率，从而延长整体的处理响应过程。

关于实际生产方面的设计实现，回头再进行总结。