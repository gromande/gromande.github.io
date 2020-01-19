---
layout: post
title: 'OpenDJ Java SDK Testing: Part 1'
date: 2016-05-13 15:50:22.000000000 -05:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Forgerock
- OpenDJ
tags:
- java
- opendj-sdk
- unit-testing
meta:
  _edit_last: '1'
  _wpas_done_all: '1'
  _jetpack_dont_email_post_to_subs: '1'
author:
  login: gromanwp
  email: guillermo.roman.dearagon@gmail.com
  display_name: Guillermo Roman
  first_name: Guillermo
  last_name: Roman
permalink: "/opendj-java-sdk-testing-part-1/"
---
If you have ever used the [OpenDJ Java SDK](https://backstage.forgerock.com/#!/docs/opendj-ldap-sdk/2.6.11/sdk-release-notes) to connect to and OpenDJ Directory Server from your application chances are that you've asked yourself&nbsp;the following question: how the hell do I test this?

Yes, you can mock most external objects and even spin up an external OpenDJ server for your integration tests, but you wonder if there is &nbsp;a cleaner and more portable approach.

In this post I will talk about using an **In-memory Backend** &nbsp;that will greatly simplify your unit test cases while&nbsp;letting you increase your test coverage. I am currently experimenting with OpenDJ Docker images for integration testing and&nbsp;I plan to publish my approach in a future post.  
<!--more-->

Let's say you have implemented a Data Access Object (DAO) that uses the OpenDJ SDK to interact with objects stored in an LDAP server. The SDK reduces most of the boilerplate of dealing with connections, entries, etc...an that's great but writing unit tests for your DAO classes can be a painful task.

Fortunately, the SDK lets you instantiate a special type of **Connection Factory** that instead of trying to connect to an external Directory Server will connect locally to a backend that resides in memory. You can think of this as a lightweight LDAP server running in memory.

This is how you would typically create a standard LDAP connection factory:

```
ConnectionFactory connectionFactory = new LDAPConnectionFactory("ldap.example.com", 1389);
```

And this is how you can create an In-memory Connection factory:

```
//Create empty memory backend MemoryBackend backend = new MemoryBackend(); //Initialize connection factory ConnectionFactory factory = Connections.newInternalConnectionFactory( Connections.newServerConnectionFactory(backend), null);
```

The ConnectionFactory interface provides a nice level of abstraction so that the classes&nbsp;using these connections (i.e. your DAO classes) won't care whether the connections are "real" (TCP connections) or "fake" (routed to a memory backend).

I've put together a simple end-to-end test to illustrate how easy it is:

```
String[] ldifEntries = new String[] { "dn: dc=groman,dc=uim", "objectClass: domain", "objectClass: top", "dc: groman", "", "dn: ou=People,dc=groman,dc=uim", "objectClass: organizationalunit", "objectClass: top", "ou: People" }; try { //Set up memory backend EntryReader entryReader = new LDIFEntryReader(ldifEntries); MemoryBackend backend = new MemoryBackend(entryReader); //Initialize connection factory ConnectionFactory factory = Connections.newInternalConnectionFactory(Connections.newServerConnectionFactory(backend), null); //Get a connection Connection connection = factory.getConnection(); //Add new entry String userDN = "uid=jdoe,ou=People,dc=groman,dc=uim"; Entry userEntry = new LinkedHashMapEntry(userDN); userEntry.addAttribute("objectClass", "top", "person"); userEntry.addAttribute("uid", "jdoe"); userEntry.addAttribute("cn", "John Doe"); userEntry.addAttribute("sn", "Doe"); Result addResult = connection.add(userEntry); assertTrue(addResult.isSuccess()); //Search for new user SearchRequest request = Requests.newSearchRequest( "ou=People,dc=groman,dc=uim", SearchScope.SINGLE\_LEVEL, "(uid=user)"); SearchResultEntry result = connection.searchSingleEntry(request); //Verify assertEquals(result.getName().toString(), userDN); assertEquals(result.getAttribute("uid").firstValueAsString(), "jdoe"); assertEquals(result.getAttribute("cn").firstValueAsString(), "John Doe"); assertEquals(result.getAttribute("sn").firstValueAsString(), "Doe"); } catch (Exception e) { e.printStackTrace(); fail(e.getMessage()); }
```

Notice how in this example I decided to initialize the MemoryBackend with some entries.

You could use the factory pattern or a dependency injection framework such as [Spring IoC](http://docs.spring.io/autorepo/docs/spring/3.2.x/spring-framework-reference/html/beans.html)&nbsp;or [Google Guice](https://github.com/google/guice) to include this approach in your testing strategy with minimum impact to your runtime code.

Finally, I would like to mention that even though you can use a Memory Backend to test most of your code it lacks certain features such as Schema checking so your units tests should always be accompanied by solid integration test cases as well. More about integration testing in the next post.

Happy testing!

