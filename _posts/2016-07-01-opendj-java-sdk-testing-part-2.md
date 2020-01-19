---
layout: post
title: 'OpenDJ Java SDK Testing: Part 2'
date: 2016-07-01 21:34:02.000000000 -05:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Forgerock
- Java
- OpenDJ
- Testing
tags:
- junit
- maven
- opendj-sdk
- unit-testing
meta:
  _edit_last: '1'
  _publicize_twitter_user: "@GuilleRoman"
  _wpas_done_all: '1'
  _jetpack_dont_email_post_to_subs: '1'
author:
  login: gromanwp
  email: guillermo.roman.dearagon@gmail.com
  display_name: Guillermo Roman
  first_name: Guillermo
  last_name: Roman
permalink: "/opendj-java-sdk-testing-part-2/"
---
<p>A few weeks back I decided to write a blog post on how to efficiently test a Java application that uses the OpenDJ SDK to connect to an LDAP store (read post <a href="http://guillermo-roman.com/opendj-java-sdk-testing-part-1/">here</a>). Since the scope was so big I had to break it down into two smaller posts. In this second part I will walk you through a sample maven-based application written in Java that uses <a href="https://www.docker.com/">Docker</a> for integration testing.</p>
<p>The following diagram illustrates the application build process:</p>
<p><img class="alignnone size-large wp-image-180" src="{{ site.baseurl }}/assets/images/maven-sldc-1024x413.png" alt="app-sdlc" width="1024" height="413" /></p>
<p>The sample app follows the standard <a href="https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html">Maven Build Life Cycle</a> but it adds a couple of phases to it so that we can build, run and destroy a Docker container before the <em>Integration Test Phase</em>.</p>
<p><!--more--></p>
<h1>About the Sample App</h1>
<p>You can download the sample application from my <a href="https://github.com/gromande/opendj-sample">Github repo</a>.</p>
<p>The application uses the <a href="https://backstage.forgerock.com/#!/docs/opendj-ldap-sdk/2.6.11/sdk-release-notes">OpenDJ Java SDK</a> to retrieve User objects from a Directory Server and perform operations on them (create, update, and delete). The code is pretty straight forward so feel free to explore it if you want to understand how it works. However, in this post I am just going to focus on testing the <em>Person Data Access Object</em> (<em>PersonDAO.java</em>). The interface is self-explanatory:</p>
<pre class="lang:java decode:true ">public interface PersonDAO {
    
    void addPerson(Person person);
    public Person getPerson(String username);
    public void updatePerson(Person person);
    public void deletePerson(String username);

}</pre>
<h1>Unit vs Integration Testing</h1>
<p>The sample app uses <a href="http://junit.org/junit4/">JUnit</a> for both unit and integration testing. To distinguish between these two types of test classes we will use <a href="http://junit.org/junit4/javadoc/4.12/org/junit/experimental/categories/Categories.html">JUnit categories</a>. This will also allow us to link each test to a different build phase.</p>
<p>The <em>IntegrationTest</em> interface is an empty interface that will be used as a marker class:</p>
<pre class="lang:java decode:true">public interface IntegrationTest {
}</pre>
<p>Use the following annotation to add a test class to the <em>IntegrationTest</em> category:</p>
<pre class="lang:java decode:true ">@Category(IntegrationTest.class)
public class MyIntegrationTestCase {
    //Your tests here
}</pre>
<p>Now that you know how to mark your tests we need to tell the <a href="http://maven.apache.org/surefire/maven-surefire-plugin/">maven-surefire-plugin</a> to skip all the annotated classes during the test phase so that only unit tests get executed:</p>
<pre class="lang:default decode:true">&lt;plugin&gt;
    &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;
    &lt;artifactId&gt;maven-surefire-plugin&lt;/artifactId&gt;
    &lt;version&gt;2.19&lt;/version&gt;
    &lt;!-- Exclude integration tests --&gt;
    &lt;configuration&gt;
        &lt;excludedGroups&gt;com.groman.opendj.dao.IntegrationTest&lt;/excludedGroups&gt;
    &lt;/configuration&gt;
&lt;/plugin&gt;</pre>
<p>Similarly we need to configure the <a href="http://maven.apache.org/surefire/maven-failsafe-plugin/">maven-failsafe-plugin</a> to only execute annotated classes during the integration-test phase:</p>
<pre class="lang:default decode:true">&lt;plugin&gt;
    &lt;groupId&gt;org.apache.maven.plugins&lt;/groupId&gt;
    &lt;artifactId&gt;maven-failsafe-plugin&lt;/artifactId&gt;
    &lt;version&gt;2.19&lt;/version&gt;
    &lt;!-- Only run integration tests --&gt;
    &lt;configuration&gt;
        &lt;groups&gt;com.groman.opendj.dao.IntegrationTest&lt;/groups&gt;
    &lt;/configuration&gt;
    &lt;executions&gt;
        &lt;execution&gt;
	    &lt;goals&gt;
		&lt;goal&gt;integration-test&lt;/goal&gt;
	    &lt;/goals&gt;
	    &lt;configuration&gt;
		&lt;includes&gt;
		    &lt;include&gt;**/*.class&lt;/include&gt;
		&lt;/includes&gt;
	    &lt;/configuration&gt;
	&lt;/execution&gt;
    &lt;/executions&gt;
&lt;/plugin&gt;</pre>
<p>The sample application only has one unit test  (<em>PersonDAOUnitTestCase</em>) and one integration test (<em>PersonDAOIntegrationTestCase</em>). Both classes extend from a base class called <em>AbstractPersonDAOTestCase</em> that implements most of the logic. The only difference between these two classes is how they initialize the LDAP connection. While <em>PersonDAOUnitTestCase</em> uses an in-memory backend, <em>PersonDAOIntegrationTestCase</em> establishes a "real" connection to an external LDAP server (in this case it will connect to a Docker container).</p>
<h1>Sample Unit Test</h1>
<p><em>PersonDAOUnitTestCase</em> makes use of the Memory Backend provided by the OpenDJ SDK. In other words, it won't actually connect to an external LDAP server. In <a href="http://guillermo-roman.com/opendj-java-sdk-testing-part-1/">Part 1</a> I talked about this feature extensively.</p>
<h1>Sample Integration Test</h1>
<p>For integration testing the idea is to spin up a Docker container running OpenDJ, execute the integration tests, and stop the container when the tests are done. The question is, how can we integrate this process into the maven build lifecycle? Well, we could write our own custom plugin...Luckily somebody has done all the hard work for us. This <a href="https://github.com/wouterd/docker-maven-plugin">docker-maven-plugin</a> does exactly what we need, and it works great. Here is how it works.</p>
<p>First we need to find a place to store the <em>Dokerfile</em> and all it's dependencies such as LDIF and schema files. I will use the same artifacts from one of my <a href="http://guillermo-roman.com/opendj-integration-testing-docker-image/">previous post</a>. You can find these files in the following location:</p>
<p><img class="alignnone wp-image-185" src="{{ site.baseurl }}/assets/images/dj_sample_eclipse_screen.png" alt="project screenshot" width="332" height="187" /></p>
<p>Time to add the <em>docker-maven-plugin</em> to our <em>pom.xml</em>. The plugin will get invoked three times during the build process:</p>
<ul>
<li><strong>build-images</strong>: executes the <em>Dockerfile</em> and build the image.</li>
</ul>
<pre class="lang:default decode:true">&lt;execution&gt;
    &lt;id&gt;package&lt;/id&gt;
    &lt;goals&gt;
        &lt;goal&gt;build-images&lt;/goal&gt;
    &lt;/goals&gt;
    &lt;configuration&gt;
	&lt;images&gt;
	    &lt;image&gt;
		&lt;id&gt;opendj&lt;/id&gt;
		&lt;dockerFile&gt;${project.basedir}/src/test/resources/opendj/Dockerfile&lt;/dockerFile&gt;
	        &lt;keep&gt;true&lt;/keep&gt;
	        &lt;nameAndTag&gt;groman/opendj-test:1.0&lt;/nameAndTag&gt;
	        &lt;artifacts&gt;
		    &lt;artifact&gt;
                        &lt;file&gt;${project.basedir}/src/test/resources/opendj/opendj.zip&lt;/file&gt;
                    &lt;/artifact&gt;
		    &lt;artifact&gt;
		        &lt;file&gt;${project.basedir}/src/test/resources/opendj/99-custom-schema.ldif&lt;/file&gt;
		    &lt;/artifact&gt;
		    &lt;artifact&gt;
		         &lt;file&gt;${project.basedir}/src/test/resources/opendj/custom-dit.ldif&lt;/file&gt;
		    &lt;/artifact&gt;
		&lt;/artifacts&gt;
	    &lt;/image&gt;
	&lt;/images&gt;
     &lt;/configuration&gt;
&lt;/execution&gt;</pre>
<ul>
<li><strong>start-containers</strong>: starts the container.</li>
</ul>
<pre class="lang:default decode:true">&lt;execution&gt;
    &lt;id&gt;start&lt;/id&gt;
    &lt;goals&gt;
	&lt;goal&gt;start-containers&lt;/goal&gt;
    &lt;/goals&gt;
    &lt;configuration&gt;
        &lt;forceCleanup&gt;false&lt;/forceCleanup&gt;
	&lt;containers&gt;
	    &lt;container&gt;
	        &lt;id&gt;opendj&lt;/id&gt;
		&lt;image&gt;groman/opendj-test:1.0&lt;/image&gt;
		&lt;waitForStartup&gt;The Directory Server has started successfully&lt;/waitForStartup&gt;
	    &lt;/container&gt;
	&lt;/containers&gt;
    &lt;/configuration&gt;
&lt;/execution&gt;</pre>
<ul>
<li><strong>stop-containers</strong>: stops and destroys the container.</li>
</ul>
<pre class="lang:default decode:true">&lt;execution&gt;
    &lt;id&gt;stop&lt;/id&gt;
    &lt;goals&gt;
	&lt;goal&gt;stop-containers&lt;/goal&gt;
    &lt;/goals&gt;
&lt;/execution&gt;</pre>
<p>If you are familiar with Docker you'll know that by default it uses ephemeral ports, which means we cannot assume that the port values will always be the same. But no worries, the plugin sets all this information in a bunch of maven properties. This information will be printed out during the build:</p>
<pre class="lang:default decode:true ">[INFO] Starting container 'opendj'..
[INFO] Setting property 'docker.containers.opendj.ports.1389/tcp.host' to '192.168.99.100'
[INFO] Setting property 'docker.containers.opendj.ports.4444/tcp.host' to '192.168.99.100'
[INFO] Setting property 'docker.containers.opendj.ports.1389/tcp.port' to '32785'
[INFO] Setting property 'docker.containers.opendj.ports.1636/tcp.host' to '192.168.99.100'
[INFO] Setting property 'docker.containers.opendj.ports.4444/tcp.port' to '32783'
[INFO] Setting property 'docker.containers.opendj.ports.1636/tcp.port' to '32784'</pre>
<p>So all we need to do is grab those maven properties and turn them into Java System properties. You can do this by simply adding the following lines to the <em>pom.xml</em>:</p>
<pre class="lang:default decode:true ">&lt;systemPropertyVariables&gt;
    &lt;opendj.hostname&gt;${docker.containers.opendj.ports.1389/tcp.host}&lt;/opendj.hostname&gt;
    &lt;opendj.ldap.port&gt;${docker.containers.opendj.ports.1389/tcp.port}&lt;/opendj.ldap.port&gt;
    &lt;opendj.bindDN&gt;cn=directory manager&lt;/opendj.bindDN&gt;
    &lt;opendj.bindPassword&gt;password&lt;/opendj.bindPassword&gt;
&lt;/systemPropertyVariables&gt;</pre>
<p>Then, your Java app can retrieve the values using <em>System.getProperty()</em>:</p>
<pre class="lang:java decode:true ">String hostname = System.getProperty("opendj.hostname");
int port = Integer.parseInt(System.getProperty("opendj.ldap.port"));
String bindDN = System.getProperty("opendj.bindDN");
String bindPassword = System.getProperty("opendj.bindPassword");</pre>
<h1>Let's Run It!</h1>
<p>I know you can't wait to see all this in action so go ahead and run the following maven command to execute the unit tests:</p>
<pre class="theme:dark-terminal lang:default decode:true">mvn test

........

[INFO] --- maven-surefire-plugin:2.19:test (default-test) @ opendj-sample ---

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.groman.opendj.dao.PersonDAOUnitTestCase
[main] DEBUG com.groman.opendj.service.LdapService - Creating service
[main] DEBUG com.groman.opendj.service.LdapService - Starting service
[main] DEBUG com.groman.opendj.dao.MemoryConnectionManager - Instantiating In-Memory Connection Factory
[main] DEBUG com.groman.opendj.service.LdapService - Stopping service
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.393 sec - in com.groman.opendj.dao.PersonDAOUnitTestCase

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0


........</pre>
<p>Now invoke the 'verify' phase to execute the Integration Tests as well:</p>
<pre class="theme:dark-terminal lang:default decode:true ">mvn verify

........

[INFO] --- maven-failsafe-plugin:2.19:integration-test (default) @ opendj-sample ---

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.groman.opendj.dao.PersonDAOIntegrationTestCase [main] DEBUG com.groman.opendj.service.LdapService - Creating service [main] DEBUG com.groman.opendj.service.LdapService - Starting service [main] DEBUG com.groman.opendj.dao.SimpleConnectionManager - Setting up Simple LDAP Connection factory: LDAPConnectionFactory(192.168.99.100:32788) [main] DEBUG com.groman.opendj.service.LdapService - Stopping service Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.969 sec - in com.groman.opendj.dao.PersonDAOIntegrationTestCase Results : Tests run: 1, Failures: 0, Errors: 0, Skipped: 0 ........

And that concludes this series of blog posts. Hope you find this useful!

