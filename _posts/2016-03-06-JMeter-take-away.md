Take aways about JMeter
=========

In JMeter Junit sampler,

**Each loop will call:**

  * the @Before setup method
  * the @Test method

(No constructor nor static method are called for each loop)  
This means the loops reuse the same object and keep calling it’s test and setup methods.

**Each thread will call:**

  * the test class constructor

(No static method is called for each thread)
This means each thread will use its own object.

**static method are called only one time***  
This means the threads share the same JVM. Classes are loaded only once.