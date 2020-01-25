---
layout: post
title: Unit Testing Using Groovy, JUnit, and Maven
date: 2016-03-07 17:27:39.000000000 -06:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
- Testing
tags:
- groovy
- junit
- maven
meta:
  _edit_last: '1'
  _jetpack_dont_email_post_to_subs: '1'
author:
  login: gromanwp
  email: guillermo.roman.dearagon@gmail.com
  display_name: Guillermo Roman
  first_name: Guillermo
  last_name: Roman
permalink: "/unit-testing-using-groovy-junit-and-maven/"
---
<p>One of my clients had to use Groovy scripts to synchronize thousands of user identities between multiple data sources. When I started working on the project the development process looked similar to this:</p>
<p><img class=" wp-image-60 aligncenter" src="{{ site.baseurl }}/assets/images/groovy-maven-junit-1-1024x458.png" alt="groovy-maven-junit-1" width="520" height="233" /></p>
<p>It's obvious that this process was far from being efficient. It was very time consuming and, most importantly, errors were caught too late in the development cycle making them very expensive to fix.</p>
<p><!--more--></p>
<p>Over the years, I've learned to ask myself the following question over and over again: what can I do to make this process more efficient? The answer in this case was simple:</p>
<blockquote><p>Test earlier. Test better.</p></blockquote>

<!--more-->

<p>So I decided to introduce a new testing phase earlier in the process:</p>
<p><img class="size-large wp-image-61 aligncenter" src="{{ site.baseurl }}/assets/images/groovy-maven-junit-2-1024x409.png" alt="groovy-maven-junit-2" width="1024" height="409" /></p>
<p>I had to tackle some problems along the way so I decided to write a blog post about it.</p>
<h2><strong>Pre-requisites</strong></h2>
<p>For this example we will be using the following components:</p>
<ul>
<li><a href="http://www.groovy-lang.org/">Groovy</a> 2.3 or higher (you can also use older versions of Groovy. Read notes at the end of the post).</li>
<li><a href="http://junit.org/">JUnit3/4</a>.</li>
<li><a href="https://maven.apache.org/">Maven</a>.</li>
<li><a href="https://groovy.github.io/gmaven/groovy-maven-plugin/">gmaven-plugin</a>.</li>
</ul>
<p>The source code for the sample maven project is available on <a href="https://github.com/gromande/groovy-junit-maven-example">github</a>.</p>
<h2><strong>Project Dependencies</strong></h2>
<p>First, you need to add the following dependencies to your pom.xml:</p>
<pre class="lang:xhtml decode:true ">&lt;dependency&gt;
    &lt;groupId&gt;org.codehaus.groovy&lt;/groupId&gt;
    &lt;artifactId&gt;groovy-all&lt;/artifactId&gt;
    &lt;version&gt;2.4.5&lt;/version&gt;
&lt;/dependency&gt;
&lt;dependency&gt;
    &lt;groupId&gt;junit&lt;/groupId&gt;
    &lt;artifactId&gt;junit&lt;/artifactId&gt;
    &lt;version&gt;4.12&lt;/version&gt;
    &lt;scope&gt;test&lt;/scope&gt;
&lt;/dependency&gt;</pre>
<p>Then, we need to instruct maven to compile our Groovy source files (if any) and run the JUnit test cases during the build process. The only thing we need to do is add the gmaven-plugin to the pom.xml as follows:</p>
<pre class="lang:xhtml decode:true">&lt;plugin&gt;
   &lt;groupId&gt;org.codehaus.gmaven&lt;/groupId&gt;
   &lt;artifactId&gt;gmaven-plugin&lt;/artifactId&gt;
   &lt;version&gt;1.5&lt;/version&gt;
   &lt;executions&gt;
   	&lt;execution&gt;
            &lt;configuration&gt;
		&lt;providerSelection&gt;2.0&lt;/providerSelection&gt;
	    &lt;/configuration&gt;
	    &lt;goals&gt;
	        &lt;goal&gt;generateStubs&lt;/goal&gt;
		&lt;goal&gt;compile&lt;/goal&gt;
		&lt;goal&gt;generateTestStubs&lt;/goal&gt;
		&lt;goal&gt;testCompile&lt;/goal&gt;
	    &lt;/goals&gt;
	&lt;/execution&gt;
    &lt;/executions&gt;
&lt;/plugin&gt;</pre>
<p>By default, the plugin expects all the Groovy source files to be located under <em>"src/main/groovy"</em> and the Groovy test files under <em>"src/test/groovy"</em>.</p>
<h2><strong>Sample Project</strong></h2>
<p>Our sample application consists of a single Groovy source file named <em>"MyGroovyClass.groovy"</em>. This file contains the class (<em>MyGroovyClass</em>) that we want to test.</p>
<p><img class="alignnone wp-image-56" src="{{ site.baseurl }}/assets/images/screenshot1-300x233.png" alt="screenshot1" width="295" height="229" /></p>
<p>I decided to add two different flavors of the same test, one for each JUnit version, to clarify the differences between the two. When using <a href="http://groovy-lang.org/testing.html#_junit_3">JUnit 3</a> you have to extend <em>GroovyTestCase</em> which, besides telling Groovy to run the class as a test, it also offers additional functionality, such as assertion methods, that will come in handy when writing your tests. Here are the contents of <em>MyGroovyClassJUnit3Test.groovy</em>:</p>
<pre class="lang:java decode:true ">import MyGroovyClass
class MyGroovyClassJunit3Test extends GroovyTestCase {

    void setUp() {
        println "Inside setup()"
    }

    void test1() {
        println "Running test 1"
        MyGroovyClass my = MyGroovyClass.newInstance()
        assertEquals("Hello", my.method1())
    }

    void test2() {
        println "Running test 2"
        MyGroovyClass my = MyGroovyClass.newInstance()
        assertEquals("Bye", my.method2())
    }

    void tearDown() {
        println "Inside tearDown()"
    }
}</pre>
<p>Take a look at the <a href="http://docs.groovy-lang.org/latest/html/api/groovy/util/GroovyTestCase.html">GroovyTestCase API docs</a> for a detailed explanation of all the assertion methods.</p>
<p>If you decide to use <a href="http://groovy-lang.org/testing.html#_junit_4">JUnit 4</a>, you can directly use standard JUnit annotations (<em>@Before, @After, @Test,</em> etc...). However, you'll have to import <em>GroovyAssert</em> to use assertion methods in your tests:</p>
<pre class="lang:java decode:true">import org.junit.*
import MyGroovyClass
import static groovy.test.GroovyAssert.*

class MyGroovyClassJunit4Test {

    @BeforeClass
    static void setUpBeforeClass() {
        println "Inside setUpBeforeClass()"
    }

    @Before
    void setUp() {
        println "Inside setUp()"
    }

    @Test
    void myTest1() {
        println "Running test 1"
        MyGroovyClass my = MyGroovyClass.newInstance() 
        assertEquals("Hello", my.method1())
    }

    @Test
    void myTest2() {
        println "Running test 2"
        MyGroovyClass my = MyGroovyClass.newInstance()
        assertEquals("Bye", my.method2())
    }

    @After
    void tearDown() {
        println "Inside tearDown()"
    }

    @AfterClass
    static void tearDownAfterClass() {
        println "Inside tearDownAfterClass()"
    }
}
</pre>
<p>Another thing to notice is that JUnit 4 lets you use the <em>@BeforeClass</em> and <em>@AfterClass</em> annotations so that you can execute some code before and after all the tests in the class have been run (this feature is not supported in JUnit 3). This is different than using the <em>@Before</em> and <em>@After</em> annotations (or the equivalent <em>setUp()</em> and <em>tearDown()</em> methods in JUnit 3). The methods annotated with @Before and @After will execute after each test is run.</p>
<h2><strong>Run the Test Cases</strong></h2>
<p>To execute all the test cases run the following maven command:</p>
<pre class="theme:dark-terminal lang:sh decode:true ">mvn clean package</pre>
<p>You should see an output similar to this:</p>
<pre class="theme:dark-terminal lang:default decode:true ">-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running MyGroovyClassJunit3Test Inside setup() Running test 1 Inside tearDown() Inside setup() Running test 2 Inside tearDown() Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.412 sec Running MyGroovyClassJunit4Test Inside setUpBeforeClass() Inside setUp() Running test 1 Inside tearDown() Inside setUp() Running test 2 Inside tearDown() Inside tearDownAfterClass() Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.05 sec Results : Tests run: 4, Failures: 0, Errors: 0, Skipped: 0

You can also run individual test cases by passing the test name as a maven property:

```
mvn clean test -Dtest=MyGroovyClassJunit3Test
```

## **Final Notes**

Regular Java classes can also be tested from your Groovy test files. Just place you classes in "_src/main/java_" as usual and import them in your Groovy test classes. Take a look at the following examples&nbsp;for more details about how to test Java classes:&nbsp;_MyJavaClassJunit3Test.groovy_ and&nbsp;_MyJavaClassJunit4Test.groovy_.

The _GroovyAssert_ class implementation was first added to Groovy 2.3. So if you are&nbsp;using Groovy 2.2 or older you will have to&nbsp;import&nbsp;_org.junit.Assert_ directly&nbsp;to&nbsp;obtain similar results.

