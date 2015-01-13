---
layout: post
title: "Spring unit test with junit  prolems"
description: "Spring unit test with junit  prolems"
category: [springmvc]
tags: [springmvc, java]
---

---------------------------------------

####JUNIT not work

**Descrption**

        java.lang.NoClassDefFoundError: org/junit/runners/model/MultipleFailureException
            at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.withAfterClasses(SpringJUnit4ClassRunner.java:188)
            at org.junit.runners.ParentRunner.classBlock(ParentRunner.java:145)
            at org.junit.runners.ParentRunner.run(ParentRunner.java:235)
            at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:163)
            at org.eclipse.jdt.internal.junit4.runner.JUnit4TestReference.run(JUnit4TestReference.java:50)
            at org.eclipse.jdt.internal.junit.runner.TestExecution.run(TestExecution.java:38)
            at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:467)
            at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:683)
            at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:390)
            at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.main(RemoteTestRunner.java:197)
        Caused by: java.lang.ClassNotFoundException: org.junit.runners.model.MultipleFailureException
            at java.net.URLClassLoader$1.run(Unknown Source)
            at java.net.URLClassLoader$1.run(Unknown Source)
            at java.security.AccessController.doPrivileged(Native Method)
            at java.net.URLClassLoader.findClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            ... 10 more

 **Solution**


        Right click on project in Package Explorer, go to Properties, go to Libraries tab, click on 'Add Library' button, select JUnit, click Next >. You should be able to handle it from there.


