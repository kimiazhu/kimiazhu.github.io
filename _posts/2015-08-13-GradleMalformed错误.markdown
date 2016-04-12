---
layout:     post
title:      "Gradle Malformed 错误"
date:       2015-08-13 13:18
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Java
    - Gradle
---

此错误在Windows上会出现，Mac上没有出现过。原因是某些文件名编码问题导致。根据文件的位置可能在报错位置上有不同。

解决方法是：

在$GRADLE_HOME\bin\gradle（Windows 下是%GRADLE_HOME%\bin\gradle.bat）配置：

	set DEFAULT_JVM_OPTS="-Dfile.encoding=UTF-8"
	

出错内容大致如下：

`gralde log: `

	PS E:\xsj\xgsdk2-portal> gradle build --stacktrace
	:compileJava UP-TO-DATE
	:processResources UP-TO-DATE
	:classes UP-TO-DATE
	:jar
	:findMainClass
	:startScripts
	:distTar
	:distZip
	:bootRepackage FAILED
	
	FAILURE: Build failed with an exception.
	
	* What went wrong:
	Execution failed for task ':bootRepackage'.
	> MALFORMED
	
	* Try:
	Run with --info or --debug option to get more log output.
	
	* Exception is:
	org.gradle.api.tasks.TaskExecutionException: Execution failed for task ':bootRepackage'.
	        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:69)
	        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.execute(ExecuteActionsTaskExecuter.java:46)
	        at org.gradle.api.internal.tasks.execution.PostExecutionAnalysisTaskExecuter.execute(PostExecutionAnalysisTaskExecuter.java:35)
	        at org.gradle.api.internal.tasks.execution.SkipUpToDateTaskExecuter.execute(SkipUpToDateTaskExecuter.java:64)
	        at org.gradle.api.internal.tasks.execution.ValidatingTaskExecuter.execute(ValidatingTaskExecuter.java:58)
	        at org.gradle.api.internal.tasks.execution.SkipEmptySourceFilesTaskExecuter.execute(SkipEmptySourceFilesTaskExecuter.java:42)
	        at org.gradle.api.internal.tasks.execution.SkipTaskWithNoActionsExecuter.execute(SkipTaskWithNoActionsExecuter.java:52)
	        at org.gradle.api.internal.tasks.execution.SkipOnlyIfTaskExecuter.execute(SkipOnlyIfTaskExecuter.java:53)
	        at org.gradle.api.internal.tasks.execution.ExecuteAtMostOnceTaskExecuter.execute(ExecuteAtMostOnceTaskExecuter.java:43)
	        at org.gradle.api.internal.AbstractTask.executeWithoutThrowingTaskFailure(AbstractTask.java:310)
	        at org.gradle.execution.taskgraph.AbstractTaskPlanExecutor$TaskExecutorWorker.executeTask(AbstractTaskPlanExecutor.java:79)
	        at org.gradle.execution.taskgraph.AbstractTaskPlanExecutor$TaskExecutorWorker.processTask(AbstractTaskPlanExecutor.java:63)
	        at org.gradle.execution.taskgraph.AbstractTaskPlanExecutor$TaskExecutorWorker.run(AbstractTaskPlanExecutor.java:51)
	        at org.gradle.execution.taskgraph.DefaultTaskPlanExecutor.process(DefaultTaskPlanExecutor.java:23)
	        at org.gradle.execution.taskgraph.DefaultTaskGraphExecuter.execute(DefaultTaskGraphExecuter.java:88)
	        at org.gradle.execution.SelectedTaskExecutionAction.execute(SelectedTaskExecutionAction.java:37)
	        at org.gradle.execution.DefaultBuildExecuter.execute(DefaultBuildExecuter.java:62)
	        at org.gradle.execution.DefaultBuildExecuter.access$200(DefaultBuildExecuter.java:23)
	        at org.gradle.execution.DefaultBuildExecuter$2.proceed(DefaultBuildExecuter.java:68)
	        at org.gradle.execution.DryRunBuildExecutionAction.execute(DryRunBuildExecutionAction.java:32)
	        at org.gradle.execution.DefaultBuildExecuter.execute(DefaultBuildExecuter.java:62)
	        at org.gradle.execution.DefaultBuildExecuter.execute(DefaultBuildExecuter.java:55)
	        at org.gradle.initialization.DefaultGradleLauncher.doBuildStages(DefaultGradleLauncher.java:149)
	        at org.gradle.initialization.DefaultGradleLauncher.doBuild(DefaultGradleLauncher.java:106)
	        at org.gradle.initialization.DefaultGradleLauncher.run(DefaultGradleLauncher.java:86)
	        at org.gradle.launcher.exec.InProcessBuildActionExecuter$DefaultBuildController.run(InProcessBuildActionExecuter.java:90)
	        at org.gradle.tooling.internal.provider.ExecuteBuildActionRunner.run(ExecuteBuildActionRunner.java:28)
	        at org.gradle.launcher.exec.ChainingBuildActionRunner.run(ChainingBuildActionRunner.java:35)
	        at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:41)
	        at org.gradle.launcher.exec.InProcessBuildActionExecuter.execute(InProcessBuildActionExecuter.java:28)
	        at org.gradle.launcher.exec.DaemonUsageSuggestingBuildActionExecuter.execute(DaemonUsageSuggestingBuildActionExecuter.java:50)
	        at org.gradle.launcher.exec.DaemonUsageSuggestingBuildActionExecuter.execute(DaemonUsageSuggestingBuildActionExecuter.java:27)
	        at org.gradle.launcher.cli.RunBuildAction.run(RunBuildAction.java:40)
	        at org.gradle.internal.Actions$RunnableActionAdapter.execute(Actions.java:169)
	        at org.gradle.launcher.cli.CommandLineActionFactory$ParseAndBuildAction.execute(CommandLineActionFactory.java:237)
	        at org.gradle.launcher.cli.CommandLineActionFactory$ParseAndBuildAction.execute(CommandLineActionFactory.java:210)
	        at org.gradle.launcher.cli.JavaRuntimeValidationAction.execute(JavaRuntimeValidationAction.java:35)
	        at org.gradle.launcher.cli.JavaRuntimeValidationAction.execute(JavaRuntimeValidationAction.java:24)
	        at org.gradle.launcher.cli.CommandLineActionFactory$WithLogging.execute(CommandLineActionFactory.java:206)
	        at org.gradle.launcher.cli.CommandLineActionFactory$WithLogging.execute(CommandLineActionFactory.java:169)
	        at org.gradle.launcher.cli.ExceptionReportingAction.execute(ExceptionReportingAction.java:33)
	        at org.gradle.launcher.cli.ExceptionReportingAction.execute(ExceptionReportingAction.java:22)
	        at org.gradle.launcher.Main.doAction(Main.java:33)
	        at org.gradle.launcher.bootstrap.EntryPoint.run(EntryPoint.java:45)
	        at org.gradle.launcher.bootstrap.ProcessBootstrap.runNoExit(ProcessBootstrap.java:54)
	        at org.gradle.launcher.bootstrap.ProcessBootstrap.run(ProcessBootstrap.java:35)
	        at org.gradle.launcher.GradleMain.main(GradleMain.java:23)
	Caused by: java.lang.IllegalArgumentException: MALFORMED
	        at org.springframework.boot.loader.tools.JarWriter.writeEntries(JarWriter.java:92)
	        at org.springframework.boot.loader.tools.Repackager.repackage(Repackager.java:179)
	        at org.springframework.boot.loader.tools.Repackager.repackage(Repackager.java:130)
	        at org.springframework.boot.gradle.repackage.RepackageTask$RepackageAction.repackage(RepackageTask.java:173)
	        at org.springframework.boot.gradle.repackage.RepackageTask$RepackageAction.execute(RepackageTask.java:138)
	        at org.springframework.boot.gradle.repackage.RepackageTask$RepackageAction.execute(RepackageTask.java:1)
	        at org.gradle.internal.Actions$FilteredAction.execute(Actions.java:201)
	        at org.gradle.api.internal.DefaultDomainObjectCollection.all(DefaultDomainObjectCollection.java:110)
	        at org.gradle.api.internal.DefaultDomainObjectCollection.withType(DefaultDomainObjectCollection.java:120)
	        at org.springframework.boot.gradle.repackage.RepackageTask.repackage(RepackageTask.java:89)
	        at org.gradle.internal.reflect.JavaMethod.invoke(JavaMethod.java:75)
	        at org.gradle.api.internal.project.taskfactory.AnnotationProcessingTaskFactory$StandardTaskAction.doExecute(AnnotationProcessingTaskFactory.java:226)
	        at org.gradle.api.internal.project.taskfactory.AnnotationProcessingTaskFactory$StandardTaskAction.execute(AnnotationProcessingTaskFactory.java:219)
	        at org.gradle.api.internal.project.taskfactory.AnnotationProcessingTaskFactory$StandardTaskAction.execute(AnnotationProcessingTaskFactory.java:208)
	        at org.gradle.api.internal.AbstractTask$TaskActionWrapper.execute(AbstractTask.java:589)
	        at org.gradle.api.internal.AbstractTask$TaskActionWrapper.execute(AbstractTask.java:572)
	        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeAction(ExecuteActionsTaskExecuter.java:80)
	        at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.executeActions(ExecuteActionsTaskExecuter.java:61)
	        ... 46 more
	
	
	BUILD FAILED
	
	Total time: 7.677 secs