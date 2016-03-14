---
layout: post
title:  "JMeter, thread and loop"
date:   2016-03-06 17:07:47 +0900
categories: jmeter
---

When doing performance test with JMeter and Junit sampler. I wonder what exact is done by the thread and loop. 
The [JMeter document](http://jmeter.apache.org/usermanual/test_plan.html), says

_Each thread will execute the test plan in its entirety and completely independently of other test threads._

(loop is about) _number of times to execute the test_

But it's still not clear to me. And the _entirety and completely independently_ part is wrong. The threads on the same server share the same JVM, therefore share the same `class` and the static stuff in it. So I did the following same test to see what does the thread and loop do.


Conslusion first, in JMeter Junit sampler,

Each **loop** will call:

  * the @Before setup method
  * the @Test method

(No constructor nor static method are called for each loop)  
This means the loops reuse the same object and keep calling it’s test and setup methods.

Each **thread** will call:

  * the test class constructor

(No static method is called for each thread)
This means each thread will use its own object.

**static method are called only one time***  
This means the threads share the same JVM. Classes are loaded only once.


Code
-----------

Following are the code and result.  
Tested in JMeter 2.13 on Ubuntu 14.04  


```java
public class Foo extends TestCase {
    private static  Bar staticBar;

    static{
        staticBar = new Bar();
        System.out.println("***********current thread:" + Thread.currentThread() + ". static is called. staticBar:" + staticBar);
    }

    private Bar memberBar;
    private Bar memberBar2;

    public Foo() {
        memberBar = new Bar();
        System.out.println(
            "***********current thread:" + Thread.currentThread() + ". Constructor is called. memberBar:" + memberBar + ". staticBar:" + staticBar);

    }

    @Before
    public void setUp(){
        memberBar2 = new Bar();
        System.out.println(
            "***********current thread:" + Thread.currentThread() + ". setup is called. memberBar:" + memberBar + ". memberBar2:" + memberBar2
                + ". staticBar:" + staticBar);
    }

    @Test
    public void testFoo(){
        System.out.println(
            "***********current thread:" + Thread.currentThread() + ". Test is called. memberBar:" + memberBar + ". memberBar2:" + memberBar2
                + ". staticBar:" + staticBar);
    }

    static class Bar{

    }
}
```
[On github](https://github.com/yetsun/JMeterUnitTest/blob/master/src/test/Foo.java)

```xml

<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="2.8" jmeter="2.13 r1665067">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="Test Plan" enabled="true">
      <stringProp name="TestPlan.comments"></stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      <elementProp name="TestPlan.user_defined_variables" elementType="Arguments" guiclass="ArgumentsPanel" testclass="Arguments" testname="User Defined Variables" enabled="true">
        <collectionProp name="Arguments.arguments"/>
      </elementProp>
      <stringProp name="TestPlan.user_define_classpath"></stringProp>
    </TestPlan>
    <hashTree>
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Thread Group" enabled="true">
        <stringProp name="ThreadGroup.on_sample_error">continue</stringProp>
        <elementProp name="ThreadGroup.main_controller" elementType="LoopController" guiclass="LoopControlPanel" testclass="LoopController" testname="Loop Controller" enabled="true">
          <boolProp name="LoopController.continue_forever">false</boolProp>
          <stringProp name="LoopController.loops">1</stringProp>
        </elementProp>
        <stringProp name="ThreadGroup.num_threads">1</stringProp>
        <stringProp name="ThreadGroup.ramp_time">1</stringProp>
        <longProp name="ThreadGroup.start_time">1457332426000</longProp>
        <longProp name="ThreadGroup.end_time">1457332426000</longProp>
        <boolProp name="ThreadGroup.scheduler">false</boolProp>
        <stringProp name="ThreadGroup.duration"></stringProp>
        <stringProp name="ThreadGroup.delay"></stringProp>
      </ThreadGroup>
      <hashTree>
        <JUnitSampler guiclass="JUnitTestSamplerGui" testclass="JUnitSampler" testname="JUnit Request" enabled="true">
          <stringProp name="junitSampler.classname">test.Foo</stringProp>
          <stringProp name="junitsampler.constructorstring"></stringProp>
          <stringProp name="junitsampler.method">testFoo</stringProp>
          <stringProp name="junitsampler.pkg.filter"></stringProp>
          <stringProp name="junitsampler.success">Test successful</stringProp>
          <stringProp name="junitsampler.success.code">1000</stringProp>
          <stringProp name="junitsampler.failure">Test failed</stringProp>
          <stringProp name="junitsampler.failure.code">0001</stringProp>
          <stringProp name="junitsampler.error">An unexpected error occured</stringProp>
          <stringProp name="junitsampler.error.code">9999</stringProp>
          <stringProp name="junitsampler.exec.setup">false</stringProp>
          <stringProp name="junitsampler.append.error">false</stringProp>
          <stringProp name="junitsampler.append.exception">false</stringProp>
          <boolProp name="junitsampler.junit4">true</boolProp>
        </JUnitSampler>
        <hashTree/>
      </hashTree>
      <ResultCollector guiclass="TableVisualizer" testclass="ResultCollector" testname="View Results in Table" enabled="true">
        <boolProp name="ResultCollector.error_logging">false</boolProp>
        <objProp>
          <name>saveConfig</name>
          <value class="SampleSaveConfiguration">
            <time>true</time>
            <latency>true</latency>
            <timestamp>true</timestamp>
            <success>true</success>
            <label>true</label>
            <code>true</code>
            <message>true</message>
            <threadName>true</threadName>
            <dataType>true</dataType>
            <encoding>false</encoding>
            <assertions>true</assertions>
            <subresults>true</subresults>
            <responseData>false</responseData>
            <samplerData>false</samplerData>
            <xml>false</xml>
            <fieldNames>false</fieldNames>
            <responseHeaders>false</responseHeaders>
            <requestHeaders>false</requestHeaders>
            <responseDataOnError>false</responseDataOnError>
            <saveAssertionResultsFailureMessage>false</saveAssertionResultsFailureMessage>
            <assertionsResultsToSave>0</assertionsResultsToSave>
            <bytes>true</bytes>
            <threadCounts>true</threadCounts>
          </value>
        </objProp>
        <stringProp name="filename">/home/jye/work/tmp/Foo</stringProp>
      </ResultCollector>
      <hashTree/>
    </hashTree>
  </hashTree>
</jmeterTestPlan>

```  
[On github](https://github.com/yetsun/JMeterUnitTest/blob/master/jmx/Foo.jmx)  



result
-----------
when

| Settings          | Values   | 
| ----------------- | --------:|
| number of threads |  2       |
| nuber of loops    |  1       |

```
***********current thread:Thread[Thread Group 1-1,6,main]. Constructor is called. memberBar:test.Foo$Bar@7343f255. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. setup is called. memberBar:test.Foo$Bar@7343f255. memberBar2:test.Foo$Bar@71dae591. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. Test is called. memberBar:test.Foo$Bar@7343f255. memberBar2:test.Foo$Bar@71dae591. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. Constructor is called. memberBar:test.Foo$Bar@4d44e298. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. setup is called. memberBar:test.Foo$Bar@4d44e298. memberBar2:test.Foo$Bar@657890c9. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. Test is called. memberBar:test.Foo$Bar@4d44e298. memberBar2:test.Foo$Bar@657890c9. staticBar:test.Foo$Bar@7c888145
```
The constructor is called 2 times. Each for a thread.


when

| Settings          | Values   | 
| ----------------- | --------:|
| number of threads |  1       |
| nuber of loops    |  2       |

```
***********current thread:Thread[Thread Group 1-1,6,main]. Constructor is called. memberBar:test.Foo$Bar@41a68efc. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. setup is called. memberBar:test.Foo$Bar@41a68efc. memberBar2:test.Foo$Bar@1df56410. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. Test is called. memberBar:test.Foo$Bar@41a68efc. memberBar2:test.Foo$Bar@1df56410. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. setup is called. memberBar:test.Foo$Bar@41a68efc. memberBar2:test.Foo$Bar@597c65cd. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. Test is called. memberBar:test.Foo$Bar@41a68efc. memberBar2:test.Foo$Bar@597c65cd. staticBar:test.Foo$Bar@7c888145
```
The setup and test methods are called 1 time for each loop. But not the onstructor, which is only called once.



when

| Settings          | Values   | 
| ----------------- | --------:|
| number of threads |  2       |
| nuber of loops    |  3       |

```
***********current thread:Thread[Thread Group 1-1,6,main]. Constructor is called. memberBar:test.Foo$Bar@2297933. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. setup is called. memberBar:test.Foo$Bar@2297933. memberBar2:test.Foo$Bar@70d40460. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. Test is called. memberBar:test.Foo$Bar@2297933. memberBar2:test.Foo$Bar@70d40460. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. setup is called. memberBar:test.Foo$Bar@2297933. memberBar2:test.Foo$Bar@6f6b747e. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. Test is called. memberBar:test.Foo$Bar@2297933. memberBar2:test.Foo$Bar@6f6b747e. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. setup is called. memberBar:test.Foo$Bar@2297933. memberBar2:test.Foo$Bar@77ad3557. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-1,6,main]. Test is called. memberBar:test.Foo$Bar@2297933. memberBar2:test.Foo$Bar@77ad3557. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. Constructor is called. memberBar:test.Foo$Bar@7e9ce042. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. setup is called. memberBar:test.Foo$Bar@7e9ce042. memberBar2:test.Foo$Bar@6d474dfe. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. Test is called. memberBar:test.Foo$Bar@7e9ce042. memberBar2:test.Foo$Bar@6d474dfe. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. setup is called. memberBar:test.Foo$Bar@7e9ce042. memberBar2:test.Foo$Bar@287fc766. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. Test is called. memberBar:test.Foo$Bar@7e9ce042. memberBar2:test.Foo$Bar@287fc766. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. setup is called. memberBar:test.Foo$Bar@7e9ce042. memberBar2:test.Foo$Bar@5cfc0e4f. staticBar:test.Foo$Bar@7c888145
***********current thread:Thread[Thread Group 1-2,6,main]. Test is called. memberBar:test.Foo$Bar@7e9ce042. memberBar2:test.Foo$Bar@5cfc0e4f. staticBar:test.Foo$Bar@7c888145
```
This is just a validation case. The constructors are called 2 times. and setup and test methods are called 6 times (for 2 threads).


