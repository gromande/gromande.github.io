---
layout: post
title: 'OpenAM 12 Custom Authentication: Error Messages'
date: 2016-04-07 16:18:36.000000000 -05:00
password: ''
categories:
- Forgerock
- OpenAM
- Programming
tags:
- authentication
- openam
---
This is the first of a series of posts in which I plan to cover some tips and pitfalls involving the creation of custom authentication modules in&nbsp;[OpenAM 12](https://backstage.forgerock.com/#!/docs/openam). One of the most common people ask me&nbsp;is how do I customize the error messages returned by the OpenAM's authentication engine?. Today I will explain step-by-step how to add custom error messages to your authentication modules.

<!--more-->

# Prerequisites

- This approach will only work with **OpenAM 12**. In the newly released OpenAM 13 there is a simper way of achieving the same result, which I intend to cover in a future post.
- Since code changes will be required you can only apply this technique for **custom authentication modules**.

You can find the sample code [here](https://github.com/gromande/openam-auth-sample-modules).

# Some background...

In OpenAM 12 the standard way of authenticating users is by invoking the [authenticate REST endpoint](https://backstage.forgerock.com/#!/docs/openam/12.0.0/dev-guide#rest-api-auth-json):

```
curl -k --request POST \ --header "X-OpenAM-Username: demo" \ --header "X-OpenAM-Password: changeit" \ --header "Content-Type: application/json" \ https://openam.groman.com:8443/openam/json/authenticate {"tokenId":"AQIC5wM2LY4SfczIWBZ\_lU0-Ryo1Y8gIHAyZ3VN\_oTgorOg.\*AAJTSQACMDEAAlNLABQtNzYyMDcwNjgxMTgwNTAzNjIzMQ..\*","successUrl":"/sso/console"}
```

If the authentication request is successful&nbsp;(like in the above example) the response will contain the authentication token for that user (e.g "_tokenId_").

When authentication fails the endpoint will return an error message instead:

**Invalid Username**

```
curl -k --request POST \ --header "X-OpenAM-Username: invalid" \ --header "X-OpenAM-Password: changeit" \ --header "Content-Type: application/json" \ https://openam.groman.com:8443/openam/json/authenticate {"code":401,"reason":"Unauthorized","message":"Authentication Failed!!"}
```

**Invalid Password**

```
curl -k --request POST \ --header "X-OpenAM-Username: demo" \ --header "X-OpenAM-Password: invalid" \ --header "Content-Type: application/json" \ https://openam.groman.com:8443/openam/json/authenticate {"code":401,"reason":"Unauthorized","message":"Invalid Password!!"}
```

Next, I will show how you can add custom error messages to your authentication module not just for the error cases listed above (invalid username or password) but for any generic error message you might want to return to the client.

# Solution in two simple steps

First, you need to add an error authentication state to your module descriptor file (typically this file can be found under _$OPENAM\_ROOT/config/auth/default/\<MODULE\_NAME\>.xml_):

```
\<ModuleProperties moduleName="Module1" version="1.0" \> \<Callbacks length="2" order="1" timeout="600" header="Using Module1"\> \<NameCallback isRequired="true"\> \<Prompt\>Username\</Prompt\> \</NameCallback\> \<PasswordCallback echoPassword="false" \> \<Prompt\>Password\</Prompt\> \</PasswordCallback\> \</Callbacks\> \<Callbacks length="1" order="2" timeout="600" header="#ERROR MESSAGE#" error="true"\> \<NameCallback\> \<Prompt\>#THIS PROMPT WILL NEVER BE SHOWN#\</Prompt\> \</NameCallback\> \</Callbacks\> \</ModuleProperties\>
```

There are a couple of things worth mentioning&nbsp;in the Callbacks definition:

- The authentication module will replace the header value (_header="#ERROR MESSAGE#"_) with the custom error message.
- The Callbacks definition should have at least one callback. This seems to be&nbsp;unnecessary since the callback will never be used but OpenAM will throw an error if no callbacks are present.
- The new state should be marked with _error="true"_.

Once the new error state is available you'll have to do two things in order to display a custom error message from your module:

1. Replace the error state's header with your custom message.&nbsp;You can add a simple method like the one below to your module. Notice that the _substituteHeader_ method gets inherited from the&nbsp;_[AMLoginModule](https://backstage.forgerock.com/static/docs/openam/12.0.0/apidocs/)_&nbsp;base class:
```
private void setErrorMessage(String errorMessage) { substituteHeader(ERROR\_STATE, errorMessage); }
```
2. Return the authentication error state (i.e. the order assign to your state in the descriptor file) from your _process_ method after replacing the header:
```
public int process(Callback[] callbacks, int currentState) throws LoginException { //Do some stuff... if (!validUsername(username)) { setErrorMessage("Invalid username: " + username); return ERROR\_STATE; } if (!validPassword(username, password)) { setErrorMessage("Invalid password. Please try again"); return ERROR\_STATE; } //Do more stuff... }
```

In our case ERROR\_STATE should be set to "2".

After redeploying the authentication module OpenAM will return the new error messages in the HTTP response body:

**Invalid Username**

```
curl -k --request POST \ --header "X-OpenAM-Username: invalid" \ --header "X-OpenAM-Password: changeit" \ --header "Content-Type: application/json" \ https://openam.groman.com:8443/openam/json/authenticate {"code":401,"reason":"Unauthorized","message":"Invalid username: invalid"}
```

**Invalid Password**

```
curl -k --request POST \ --header "X-OpenAM-Username: demo" \ --header "X-OpenAM-Password: invalid" \ --header "Content-Type: application/json" \ https://openam.groman.com:8443/openam/json/authenticate {"code":401,"reason":"Unauthorized","message":"Invalid password. Please try again"}
```

Hope you find this useful and don't forget to leave a comment with your questions/feedback!

See you soon.

&nbsp;

