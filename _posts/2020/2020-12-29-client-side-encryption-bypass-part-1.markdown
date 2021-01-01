---
layout: "post"
title: "Client Side Encryption Bypass Part-1"
date: ""
excerpt: ""
comments: true
---

### **TL;DR**
Hi, this is [Debugger](https://github.com/bhattsameer) ready to debug JavaScript.
In this blog we will discuss the extra security layer implemented inside the application which is **encryption mechanism**. It will be a series of how to break the client side encryption and add our payload to the actual parameter and perform the application security testing. For this series we will use [DevTools](https://developers.google.com/web/tools/chrome-devtools) as our main Tool.
Below is the series of content:

Part 1: Breaking the encryption of web application and bypass OTP.</br>
Part 2: Some cool magic tricks which helps to find the encryption logic very easily.</br>
Part 3: Understanding the Logic of encryption and fuzz the parameters.</br>
their is lot more stay tune for that.

```
Debugger is you're friend.
```
### Introduction

Now every financial sector applications i.e. mobile or web, use one more security layer which is encryption mechanism so the attacker who is able to intercept the traffic through any MITM tools can not able to understand the request data. When we do pen-testing we follow some methodology, we have to test each and every parameter and request. Well as we all no attackers don't follow any rules and regulations, when they want to attack they will find a way to do it. So as keeping the mindset of an attacker, we will understand this kind of encryption mechanism, what developer thinks when they implement this? Also, what kind of mistakes they do? Why they feel putting encryption means the application is secure? What makes them think that no one can break their logic? So they hide sensitive information behind the encryption. So keeping all the above maybe some more cases in my mind, I prepared my own ""Debugging methodology"" for this, which I follow when I face this kind of scenario.

### What is JavaScript?

JavaScript is one of the most popular and widely used programming languages in the world right now, still growing more and more, big companies like [Netflix](https://www.netflix.com/in/), [PayPal](https://www.paypal.com/in/home) build an entire application around JavaScript.

JavaScript is everywhere.

For the developers it is used to build stuff but for attacker javascript is used for breaking stuff.

For more please visit: [Javascript.info](http://javascript.info/)

### What you can do with JavaScript?

As I said Javascript is everywhere.

previously javascript was only used in browser to build interactive web application, later with the support of huge community and companies i.e. google and facebook, this days you can build a complete 
1. web/mobile applications
2. Real-time networking apps (chats, video streamings)
3. command line tools
4. games
5. Desktop Application.
6. [Windows95](https://github.com/felixrieseberg/windows95)

### JavaScript vs ECMAScript?

Javascript is language and [ECMAScript](https://en.wikipedia.org/wiki/ECMAScript) is a scripting-language specification standardized by Ecma International.

### Where does JavaScript code run?

JavaScript engines (V8 for Chrome, spidermonkey for firefox etc.)
Previously we only able to run javascript inside browser only.
but later on Node was developed (which is nothing but Javascript engine outside browser).

### Debugging with DevTool:


### How the normal request and response structure looks if there is no encryption is implemented.

below is the normal request and response structure which shows that normally the parameteres are in clear text as there is no encryption is implemented.

![](/images/encryption_bypass_part1/1.png)

### How the normal request and response structure looks if there is encryption is implemented.

![](/images/encryption_bypass_part1/2.png)

In above screen shot you can see the password parameter is encrypted and the response as well.

So to test the password parameter with our paylaods we have to break the encryption and also to understand the response we need to break the encrypted response first.

### What developer thinks?

For developers they usually hide sensitive information inside the encrypted value, as they assume that the encryption means they are secure, no one can break there encryption even if the logic is default one.

Even if you highlight any issue to them, the first remidiation they will think about it encrypt the encrypted data again or encrypt the encryption key kind of chaining and they say now you can not break it.

### Breaking and Bypassing encryption:

 1. **What?**:
 
    First Question is what?
    As there are multiple ways of encryption can be implemented.
    Most common example is :
    
    1. **Encrypting the parameters:**
        i.e. some parameters value will get encrypted.
        ![](/images/encryption_bypass_part1/2.png)
        
    2. **Encrypting the whole body:**
        i.e. whole post data is encrypted, so no one can guess what are the parameters passing through this request.
 
 2. **How?**:
 
    There are lot of ways to break the encryption, but below are the steps which i personally follow to find the logic and break the encryption.
    We will understand below steps with any example:
    i.e. we have one application which asks for user email and password for authentication.
    
    ![](/images/encryption_bypass_part1/3.png)
    
    once we entered correct details it will send an OTP to registered mobile number and email address.
    
    ![](/images/encryption_bypass_part1/4.png)
    
    When you provide valid OTP you can log in, else you can not.
    
    ![](/images/encryption_bypass_part1/5.png)
    
    **So our aim is to bypass the OTP.**
 
    a.) **Understand the application and it's flow.**
    
    Always first understand the flow of application by using valid credentials (if possible) also try to understand the encryption or the request where the encryption is implemented, so that you can understand how the encryption is getting generated.
    
    Mainly focus on all the requests which triggers when you click on the submit button or interact with the application.
    
    Once you will identify that what really you wanted to break, is it a parameter which is encrypted? or the whole request body?
    great you are good to go than.
    
    For this step I am hopping you are already familier with **burpsuite tool** but it is not compulsory as we are just using burp to understand the request structure same we can do with DevTool Network option as well.
    
    Lets intercept the login request first and observe the request and response structure.
        
    ![](/images/encryption_bypass_part1/6.png)
        
    In the request *Password* parameter is encrypted and also the whole response is encrypted as well.
    
    Lets intercept the next OTP request as our aim is to bypass the OTP.
    
    ![](/images/encryption_bypass_part1/7.png)
    
    We can observe that our six digit otp is converted into an encrypted value using some logic and response from server is also something encrypted that we do not know but we can get the response in front end or in browser.
    
    i.e. below is error response when we have entered wrong OTP.
    
    ![](/images/encryption_bypass_part1/9.png)
    
    ![](/images/encryption_bypass_part1/8.png)
    
    Next we have to observe all the requests which triggers when we click on Login button.
    
    i.e. here we can see there are four main requests triggers when we click on Login and OTP:
    
    ![](/images/encryption_bypass_part1/10.png)
    
    First index.php will invoke which redirect to Validate.php and that validate.php will contains our email and encrypted password as parameter and in response we will get encrypted data.
    
    Post we will get OTP.php page where we need to fill up our OTP, and when we click on submit it will redirect to otpvalidate.php and that otpvalidate will contains our otp in encrypted format and in response we will get encrypted data.
    
    Post success we will get My_account.php which is dashboard of user who is logged in.
    
    
    b.) When you found something, ask yourself is it on client side?
    
    Now the guessing games begins :))
    
    1. As we have seen the response is in encrypted format but in browser we will get some response in clear text i.e. when the data is invalid we are getting "access denied", so lets assume that the "access denied" is there in the encrypted response hence somewhere in the client side it's got decrypted and we got the string value as an output. This will be our first guess that decryption logic is present in the client side somewhere.
    2. Next we can start with the OTP parameter, just copy that parameter value and use burp search utility and search if kind of same encrypted value you will get in any of the response, this is just a case to test if server is sending encrypted value to client side first and in next request the same encrypted value is getting used or not. The same can be done by observing every request body and response body for encrypted data.
    3. Once the above test case fail we can guess that the encryption logic somewhere in the client side.
    4. Next to confirm the encryption is on client side, lets look into the Javascript files. 
    
    c.) Look for all files.
    
    From here I am going to use DevTool only no burp is required anymore:))
    
    We are using chrome DevTool only as it provides the modification of values at run time also the snippet is the best:))
    
    
    
    d.) Want to break encryption? Or Bypass something?
    
    e.) Find the logic.
  
### Key take away:

 1. Reality........
 
 2. Obfuscation: 

Thanks for reading!!
You can follow me on twitter:
Github: 
