---
title: 使用JUnit
toc: true
date: 2018-04-07 14:00:25
categories:
- tool
tags: [Java, Tool, JUnit]
---
JUnit4依赖`Hamcrest`

Assert类
@Runwith(SpringJUnit4ClassRunner.class)
Expected Exceptions
@Test(timeout=1000) - milliseconds 

https://www.cnblogs.com/draenei/p/5103503.html
@BeforeClass        
public static void 


沿用JUnit3

@Before  
setUp()

@After
tearDown()

@BeforeClass
public static void runBeforeClass(){

}

@AfterClass
public static void runAfterClass(){
    
}
