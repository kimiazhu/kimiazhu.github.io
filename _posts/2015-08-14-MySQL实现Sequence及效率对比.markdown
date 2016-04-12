---
layout:     post
title:      "MySQL实现Sequence及效率对比"
date:       2015-08-14 16:58
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Spring Boot
    - Java
    - MySQL
    - MyBatis
---

MySQL没有Sequence，我们可以用两种方法解决：

1. 做一张独立的表，只含有一个id自增字段，模拟Sequence；
2. 写一个通用的Sequence表，一条记录表示一个Sequence，业务可共用。

我们对这两种情况提供实现和效率测试。

实现过程基于Spring Boot v1.2.5 / MyBatis v3.2 / MySQL v5.1。测试使用JUnit进行并发请求模拟。

### 一、自增字段模拟Sequence

#### 1. 建表
`SQL:`
	
	CREATE TABLE AUTOINCREMENT_SEQ
	(
	  id                        INT NOT NULL AUTO_INCREMENT,
	  PRIMARY KEY (id) 
	);

#### 3. 实体定义
`Java:`

	package info.kimiazhu.demo.model;

	public class AutoincrementSeq {

	    private int id;

	    public int getId() {
	        return id;
	    }

	    public void setId(int id) {
	        this.id = id;
	    }
	}


#### 3. Insert语句
`AutoincrementSeqMapper.xml:`

	<insert id="insert" useGeneratedKeys="true" keyProperty="id" parameterType="info.kimiazhu.demo.model.AutoincrementSeq">
        INSERT INTO AUTOINCREMENT_SEQ () VALUES ()
    </insert>

Insert语句在插入之后返回主键设入实体中，以后要使用这个ID就调用实体的get方法获取。

### 二、建一张通用的Sequence表

#### 1. 建表
这里使用了MySQL用户自定义函数来定义了三个函数：**currval**/**nextval**/**setval**。

`SQL:`

	CREATE TABLE SEQUENCE
	(
	  name                      VARCHAR(64)                    NOT NULL,
	  current_value             BIGINT                            NOT NULL,
	  step                      INT                            NOT NULL DEFAULT 1,
	  PRIMARY KEY (name) 
	);

	ALTER TABLE SEQUENCE COMMENT 'Sequence自增表';

	DROP FUNCTION IF EXISTS currval;
	DELIMITER $$
	CREATE FUNCTION currval (seq_name VARCHAR(64))
	RETURNS INTEGER
	CONTAINS SQL
	BEGIN
	  DECLARE value INTEGER;
	  SET value = 0;
	  SELECT current_value INTO value
	  FROM SEQUENCE
	  WHERE name = seq_name;
	  RETURN value;
	END $$
	DELIMITER ;

	DROP FUNCTION IF EXISTS nextval;
	DELIMITER $$
	CREATE FUNCTION nextval (seq_name VARCHAR(64))
	RETURNS INTEGER
	CONTAINS SQL
	BEGIN
	   UPDATE SEQUENCE
	   SET current_value = last_insert_id(current_value + step) 
	   WHERE name = seq_name;
	   RETURN last_insert_id();
	END;
	$$ 
	DELIMITER ;

	DROP FUNCTION IF EXISTS setval;
	DELIMITER $$
	CREATE FUNCTION setval (seq_name VARCHAR(64), value INTEGER)
	RETURNS INTEGER
	CONTAINS SQL
	BEGIN
	   UPDATE SEQUENCE
	   SET current_value = value
	   WHERE name = seq_name;
	   RETURN currval(seq_name);
	END;
	$$
	DELIMITER ;
	
#### 2. 初始化数据

以APP_ID为例，我们规定APP_ID每次自动生成，但它不是数据库表中的主键，所以我们准备使用Sequence表中的一条记录来生成。

`SQL:`

	INSERT INTO sequence(name, current_value, step) VALUES ("APP_ID", 0, 1);
	
#### 3. Mapper语句

`SequenceMapper.xml:`

	<mapper namespace="info.kimiazhu.demo.mapper.SequenceMapper">
	    <select id="nextValue" resultType="int">
	        select nextval(#{0}); 
	    </select>
	    
	    <select id="currentValue" resultType="int">
	        select currval(#{0}); 
	    </select>
	    
	    <select id="setValue" resultType="int">
	        select setval(#{0}, #{1}); 
	    </select>
	</mapper>
	
### 三、测试

测试代码使用JUnit，加入GroboUtils进行多线程并发模拟next_value请求。起100个线程，每个线程内部请求100次next_value。

#### 1. 自增表的测试用例：

`AutoincrementSeqMapperTest.java`

	package info.kimiazhu.demo.mapper;

	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.SpringApplicationConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

	import info.kimiazhu.demo.DemoApplication;
	import info.kimiazhu.demo.model.AutoincrementSeq;
	import net.sourceforge.groboutils.junit.v1.MultiThreadedTestRunner;
	import net.sourceforge.groboutils.junit.v1.TestRunnable;

	@RunWith(SpringJUnit4ClassRunner.class)
	@SpringApplicationConfiguration(classes = DemoApplication.class)
	public class AutoincrementSeqMapperTest {
	    
	    private static final Logger LOGGER = LoggerFactory.getLogger(AutoincrementSeqMapperTest.class);
	    
	    @Autowired
	    private AutoincrementSeqMapper autoincrementSeqMapper;
	    
	    @Test
	    public void testInsert() throws Exception {
	        TestRunnable runner = new TestRunnable() {
	            @Override
	            public void runTest() throws Throwable {
	                for(int i = 0; i < 100; i++) {
	                    AutoincrementSeq autoincrementSeq = new AutoincrementSeq();
	                    autoincrementSeqMapper.insert(autoincrementSeq);
	                }
	            }
	        };
	        
	        int runnerCount = 100;
	        TestRunnable[] runners = new TestRunnable[runnerCount];
	        for (int i = 0; i < runnerCount; i++) { 
	            runners[i] = runner; 
	        }
	        
	        MultiThreadedTestRunner mttr = new MultiThreadedTestRunner(runners);
	        long start =  System.currentTimeMillis();
	        try {
	            mttr.runTestRunnables();
	        } catch (Throwable t) {
	            LOGGER.error("Error occur", t);
	        }
	        LOGGER.info("elapse time: " + ((System.currentTimeMillis() - start)/1000.f) + "s");
	    }
	}
	
#### 2. Sequence表测试

`SequenceMapperTest.java`

	package info.kimiazhu.demo.mapper;

	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.SpringApplicationConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

	import info.kimiazhu.demo.DemoApplication;
	import info.kimiazhu.demo.mapper.SequenceMapper.SEQ_NAME;
	import net.sourceforge.groboutils.junit.v1.MultiThreadedTestRunner;
	import net.sourceforge.groboutils.junit.v1.TestRunnable;

	@RunWith(SpringJUnit4ClassRunner.class)
	@SpringApplicationConfiguration(classes = DemoApplication.class)
	public class SequenceMapperTest {
	    
	    private static final Logger LOGGER = LoggerFactory.getLogger(SequenceMapperTest.class);
	    
	    @Autowired
	    private SequenceMapper sequenceMapper;
	    
	    @Test
	    public void testNextValue() throws Exception {
	        TestRunnable runner = new TestRunnable() {
	            @Override
	            public void runTest() throws Throwable {
	                for(int i = 0; i < 100; i++) {
	                    sequenceMapper.nextValue(SEQ_NAME.APP_ID);
	                }
	            }
	        };
	        
	        int runnerCount = 100;
	        TestRunnable[] runners = new TestRunnable[runnerCount];
	        for (int i = 0; i < runnerCount; i++) { 
	            runners[i] = runner; 
	        }
	        
	        MultiThreadedTestRunner mttr = new MultiThreadedTestRunner(runners);
	        long start =  System.currentTimeMillis();
	        try {
	            mttr.runTestRunnables();
	        } catch (Throwable t) {
	            LOGGER.error("Error occur", t);
	        }
	        LOGGER.info("elapse time: " + ((System.currentTimeMillis() - start)/1000.f) + "s");
	    }
	}

#### 3. 测试结果

- 执行1万次next_value操作，自增表的方式消耗25s-26s
- 执行1万次next_value操作，自建Sequence表的方式消耗18s-19s
- 基本上内建函数的Sequence方式是自增表方式效率提升1/4
- 内建函数的方式并不存在并发问题。
- **<font color="red">更正，关于效率的测试是有问题的，sequence表的方式效率会比较低，需要一个高效的实现方式，我们后来做了优化，在一次性取出步长为N的长度，然后在内存中进行分配。当N条都用完之后，再从数据库Sequence表中申请。</font>**

### 三、结论

自建Sequence和内建函数的方式更为通用，当有多个业务需要自增序列的时候，一个表就可以满足需求，并且不会因为使表的记录数暴增。

独立建一张表，用自增序列模拟的方式实现较简单，不依赖数据库内建函数，迁移成本较小。但是效率相对低一些。

[示例代码](https://github.com/kimiazhu/java-playground/tree/master/mysql-sequence-demo)
