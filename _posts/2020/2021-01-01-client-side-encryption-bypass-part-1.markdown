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
Debugger is your friend.
```
### Introduction

Now every financial sector applications i.e. mobile or web, use one more security layer which is encryption mechanism so the attacker who is able to intercept the traffic through any MITM tools can not able to understand the request data. When we do pen-testing we follow some methodology, we have to test each and every parameter and request. Well as we all no attackers don't follow any rules and regulations, when they want to attack they will find a way to do it. So as keeping the mindset of an attacker, we will understand this kind of encryption mechanism, what developer thinks when they implement this? Also, what kind of mistakes they do? Why they feel putting encryption means the application is secure? What makes them think that no one can break their logic? So they hide sensitive information behind the encryption. So keeping all the above maybe some more cases in my mind, I prepared my own ""Debugging methodology"" for this, which I follow when I face this kind of scenario.

### What is JavaScript?

JavaScript is one of the most popular and widely used programming languages in the world right now, still growing more and more, big companies like [Netflix](https://www.netflix.com/in/), [PayPal](https://www.paypal.com/in/home) build an entire application around JavaScript.

JavaScript is everywhere.

For the developers it is used to build stuff but for attacker JavaScript is used for breaking stuff.

For more please visit: [JavaScript.info](http://javascript.info/)

### What you can do with JavaScript?

As I said JavaScript is everywhere.

Previously JavaScript was only used in browser to build interactive web application, later with the support of huge community and companies i.e. Google and Facebook, this day's you can build a complete -

1. Web/Mobile applications. </br>
2. Real-time networking apps (chats, video streaming).</br>
3. Command line tools.</br>
4. Games.</br>
5. Desktop Application.</br>
6. [Windows95](https://github.com/felixrieseberg/windows95).

### JavaScript vs ECMAScript?

Javascript is a language and [ECMAScript](https://en.wikipedia.org/wiki/ECMAScript) is a scripting-language specification standardized by ECMA International.

### Where does JavaScript code run?

JavaScript engines (V8 for Chrome, spider monkey for Firefox etc.)
Previously we were only able to run JavaScript inside browser only.
but later on Node was developed (which is nothing but Javascript engine outside browser).

### Debugging with DevTools:

We will not go in the basic's of DevTools, to understand the DevTools you can read this [Blog](https://blittle.github.io/chrome-dev-tools/)

### Let get into the main topic:

#### How the normal request and response structure looks if there is no encryption implemented.

Below is the normal request and response structure which shows that normally the parameters are in clear text as there is no encryption is implemented.

![](/images/encryption_bypass_part1/1.png)

### How the normal request and response structure looks if there is an encryption implemented.

![](/images/encryption_bypass_part1/2.png)

In above screen shot you can see the password parameter is encrypted and the response as well.

So to test the password parameter with our paylaods we have to break the encryption and also to understand the response we have to break the encrypted response as well.

### What developers think?

For developers they usually hide sensitive information inside the encrypted value, as they assume that the encryption means they are secure, no one can break there encryption even if the logic is default one.

Even if you highlight any issue to them, the first remidiation they will think about it encrypt the encrypted data again or encrypt the encryption key kind of chaining and they will claim that no one can break this.

### Breaking and Bypassing encryption:

 1. **What?**:
 
    As there are multiple ways of encryption can be implemented on request data.</br>
    Most common example is :
    
    1. **Encrypting the parameters:**
        i.e. some parameters value will get encrypted.</br>
        ![](/images/encryption_bypass_part1/2.png)
        
    2. **Encrypting the whole body:**
        i.e. whole post data is encrypted, so no one can guess what are the parameters passing through this request.</br>
 
 2. **How?**:
 
    There are lot of ways to break the encryption, but below are the steps which I personally follow to find the logic and break the encryption.</br>
    We will understand below steps with one example:</br>
    
    I have prepared one demo application which helps us to understand the process, you can get the docker version of it from this [link](), once you are done with the setup please continue for the next.
    
    So we have one web application which asks for user email and password for authentication.
    
    ![](/images/encryption_bypass_part1/3.png)
    
    once we entered correct details it will send an OTP to registered mobile number and email address.
    
    ![](/images/encryption_bypass_part1/4.png)
    
    When you provide valid OTP you can log in, else you will get "Access Denied".
    
    ![](/images/encryption_bypass_part1/5.png)
    
    **So our aim is to bypass the OTP after breaking the encryption logic.**
 
    a.) **Understand the application and it's flow.**
    
    Always first understand the flow of application by using valid credentials (if possible) also try to understand the encryption or the request where the encryption is implemented, so that you can understand how the encryption is getting generated.
    
    Mainly focus on all the requests which triggers when you click on the submit button or interact with the application.
    
    Once you will identify that what really you wanted to break, is it a parameter which is encrypted? or the whole request body?
    great you are good to go than.
    
    For this step I am hopping you are already familier with [**burpsuite tool**](https://portswigger.net/burp) but it is not compulsory as we are just using burp to understand the request structure same we can do with DevTools Network option as well.
    
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
    
    First login.php will invoke which redirect to Validate.php and that validate.php will contains our email and encrypted password as parameter and in response we will get encrypted data.
    
    Post we will get OTP.php page where we need to fill up our OTP, and when we click on submit it will redirect to otpvalidate.php and that otpvalidate will contains our otp in encrypted format and in response we will get encrypted data.
    
    Post success we will get My_account.php which is dashboard of user who is logged in.
    
    
    b.) When you found something, ask yourself is it on client side?
    
    Now the guessing games begins :))
    
    1. As we have seen the response is in encrypted format but in browser we will get some response in clear text i.e. when the data is invalid we are getting "access denied", so lets assume that the "access denied" is there in the encrypted response hence somewhere in the client side it's got decrypted and we got the string value as an output. This will be our first guess that decryption logic is present in the client side somewhere.
    2. Next we can start with the OTP parameter, just copy that parameter value and use burp search utility and search if kind of same encrypted value you will get in any of the response, this is just a case to test if server is sending encrypted value to client side first and in next request the same encrypted value is getting used or not. The same can be done by observing every request body and response body for encrypted data.
    3. Once the above test case fail we can guess that the encryption logic may somewhere on the client side.
    4. Next to confirm the encryption is on client side, lets look into the Javascript files. 
    
    c.) Look for all files.
    
    From here onward I am going to use DevTools only no burp is required anymore:))
    
    We are using chrome DevTools only as it provides the modification of values at run time so you can modify the value of parameters before it gets encrypted Also, the snippet feature of it is the best:))
    
    First of all, we'll have to open up the DevTools. There are multiple ways to do that:
    - Click on F12 to open it up.
    - Right click on mouse and navigate to inspect element.
    - Open it up from Developer tools section from browser settings.
    
    Image 
   
    Here in this blog, we will only talk about:
    
    1. Element: Element section contains the DOM Tree of the page you are viewing. As you hover over elements in the DOM tree, they will automatically highlight on the page.
    2. Console: It is used to run the your JavaScript code.
    3. Sources: This is the most powerful and useful feature of DevTools. All the sources called or used by the application at that moment can be found here as a source tree of application.
    
    We will talk about other tabs later on in the series.
    
    As we know the logic part will be written as a JavaScript, hence we can assume that the encryption logic is also somewhere in the JavaScript file. This is not the only case some time the JS code is used inside the same file using <script> tag.
  
    To get the list of source in DevTools navigate to sources tab and observe all the files.
    
    Image of source tab
    
    One way to indentify the logic is to read the all JS files and logic's used inside the application and understand it. But this process is really take out lot of your time and also sometimes the code is obfuscated and also it is very lengthy as well.
    So one easy way to find the logic is using keyword based search, in source tab click **Ctrl+Shift+f** and you will get one search bar, this is a global search feature of Chrome DevTools, by which you can search the text in all the files listed there in source tab.
    
    Some of the best keywords you can search for:
    encrypt
    crypt
    OTP (As the otp parameter is having encrypted value)
    password (As the password field is having encrypted value)
    RSA (Encryption type) 
    AES (Encryption type)
    key
    
    Once you search for the keyword you will get the result and when you click one of them you will be autometically redirected to that specific file and the line number.
    
    [Image source]
    
    Observe the below screen shot we found one method *logear* which contains key and perfom encryption of password.
    
    [Image source]
    
    There is also one another way using inspect element feature, navigate to the element tab in DevTools and navigate to login button in DOM tree and observe the tag and it's attribute, some time the attributes help you get the action method. Like in below screen shot: 
    we have one attribute *onClick* which contains value *logear()* and this *logear()* is method, which is responsible for encryption.
    
    [Image element]
    
    d.) Want to break encryption? Or Bypass something?
    
    
    
    
    
    e.) Find the logic.
   
### Key take away:

 1. Reality........
 
 2. Obfuscation: 

Thanks for reading!!
You can follow me on twitter:
Github: 
