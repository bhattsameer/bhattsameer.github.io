---
layout: "post"
title: "Client Side Encryption Bypass Part-2"
date: "2021-02-14 4:00"
excerpt: "In this blog we will continue from where we left in Part-1 of this client side encryption bypass series.
We will see some cool tips and tricks related to DevTools, which will be really helpful when you try break the encryption logic on your own."
comments: false
---

### **TL;DR**
Hi, this is [Debugger](https://github.com/bhattsameer) ready to debug JavaScript.  

In this blog we will continue from where we left in [Part-1](https://github.com/bhattsameer/bhattsameer.github.io/blob/main/_posts/2021/2021-01-01-client-side-encryption-bypass-part-1.markdown) of this client side encryption bypass series.   
We will see some cool tips and tricks related to [DevTools](https://developers.google.com/web/tools/chrome-devtools), which will be helpful when you try break the encryption logic on your own.  

```
 Debugger is always your BestFriend.
```

### Introduction

Before starting, I strongly suggest if you have't checked out the [part-1](https://github.com/bhattsameer/bhattsameer.github.io/blob/main/_posts/2021/2021-01-01-client-side-encryption-bypass-part-1.markdown) please read that first.  

Last time we have seen the steps to break the encryption mechanism, but the process of indentifying the encryption logic sometimes became very complex, and chances to finding the responsible method from huge list of files will be time consuming as you need to put debug points at all the places. 

![Picture1.png](/images/encryption_bypass_part2/Picture1.png)

**So are there any ways to make this process easier?**

Well, not completely. But yes there are some magic tricks which can helps us in making the process not complex much and they will help us for sure in our debugging process.  

### Let's see the magic tricks one by one:

#### 1. debug():  

 This one is built in method of devtools which helps you to debug the method directly without putting any debug points. To use this method all you need is name of the method where you want to stop the execution process and all other work will be handled by this method only. Devtool will take you to the method line number directly and let you observe the current values of all the parameters and objects in the scope section.
 
 We can use this method to stop the continue step in and step over process to reach to the specific debug place or value, we can directly put the debug method and it will stop execution at the place where we wants.
     
 **Steps to use this:**  
 For the practical purpose we can take example of website: [https://googlechrome.github.io/devtools-samples/debug-js/get-started](https://googlechrome.github.io/devtools-samples/debug-js/get-started)   
 
  a.) Navigate to "[https://googlechrome.github.io/devtools-samples/debug-js/get-started](https://googlechrome.github.io/devtools-samples/debug-js/get-started)" and observe the application behaviour.  
  
  ![debug_1](/images/encryption_bypass_part2/1.png)
  
  b.) Open the inspect element or devtool and navigate to "Source" tab. Observe the responsible javascript file and method.  
  
  ![debug_2](/images/encryption_bypass_part2/2.png)
  
  c.) Navigate to "console" section in devtool and run ***debug(method_name)***. Observe the output in console.  
  
  ![debug_3](/images/encryption_bypass_part2/3.png)
  
  d.) Now when you "click on the add number button again" the ***updateLabel*** gets executed and suddenly the debug point gets envoked, resulting the execution stops at line number 29 for you to watch the variable type "string".  
  
  ![debug_4](/images/encryption_bypass_part2/4.png)
 
 **Note:** To stop debugging the method run ***undebug(method_name)*** in console.
 
 ### 2. monitor():  

 This one is built in method of devtools which helps you to monitor the method directly without putting any debug points. To use this method all you need is name of the method, and rest all handled by monitor(), it will give you notification in console that the method is executed.
 
 We can use this method to monitor that which method is responsible for encryption from multiple encryption logics implemented in the application, this will just reduce our time to check each and every method. So we can monitor that whenever we click on submit button, which method get called out first.
     
 **Steps to use this:**  
 For the practical purpose we can take example of website: [https://googlechrome.github.io/devtools-samples/debug-js/get-started](https://googlechrome.github.io/devtools-samples/debug-js/get-started)  
 
  a.) Navigate to "[https://googlechrome.github.io/devtools-samples/debug-js/get-started](https://googlechrome.github.io/devtools-samples/debug-js/get-started)" and open the devtool console section. Post run ***monitor(method_name)*** and Observe the output in console. Keep your eyes on console as you will notifications in console only.  
  
  ![monitor_1](/images/encryption_bypass_part2/5.png)
  
  b.) Now when you "click on the add number button again" the ***updateLabel*** gets executed and you will get a notification in console.  
  
  ![monitor_2](/images/encryption_bypass_part2/6.png)
 
 **Note:** To stop monitoring the method run ***unmonitor(method_name)*** in console.
 
### 3. snippets: 
 
 This one is the most useful and cool feature of devtool. You can run your own javascript inside the DOM of current application, means you can replace the method with your own method by writing the snippet of it and execute it. 
 
 using this feature you can fuzz the each and every parameter of application even if they are encrypted as you can use the same logic of encryption implemented by the developers by just calling out the method name and decrypt the parameters, enter your payload and encrypt it again.  
 **Sounds cool? using the developers tool and developers own logic for breaking there application.:))**
 
 **Steps to use this:**  
 For the practical purpose we can take example of website: [https://googlechrome.github.io/devtools-samples/debug-js/get-started](https://googlechrome.github.io/devtools-samples/debug-js/get-started)     
 
  a.) Navigate to "[https://googlechrome.github.io/devtools-samples/debug-js/get-started](https://googlechrome.github.io/devtools-samples/debug-js/get-started)" and open up the devtool, access snippet feature of source tab. Here observe that we have written our own snippet code.  
  
  ![snippet_1](/images/encryption_bypass_part2/7.png)
  
  In the code observe that we have changed the previous ***updateLabel()*** method to argument based ***updateLabel(arg1, arg2)*** method.
  
  b.) To execute the snippet code click on "Run" at bottom or use keyboard keys "Ctrl + Enter".
 
 **Note:** This feature is available in both firefox and chrome, But I have created all my snippets on chrome only as it allows you to modify parameter's value at run time in devtools.  
 
 **I have prepared the list of devtool-snippets-forhacks, you can check it out on my [github](https://github.com/bhattsameer).**
 
 ### 4. monitor() + Snippet:
 
 We will use monitor method on our snippet, I am doing this to tell you more about monitor(). Monitor not just tells you when the method invoked but also tells you the current arguments passed to it.
 
 This one really helpful because most of time the encryption methods are like ***method_name(data,key,salt)***, and if the key is same for all encryption logic, you can directly get the key by monitoring the method.
 
 **Steps to use this:**  
 For the practical purpose we can take example of website: [https://googlechrome.github.io/devtools-samples/debug-js/get-started](https://googlechrome.github.io/devtools-samples/debug-js/get-started)  
 
  a.) Once you have executed your snippet code, go back to console and run ***monitor(method_name)***. Now when you "click on the add number button again" the updated ***updateLabel*** will gets executed and you will get a notification in console. Observe that the arguments are also in the notification.  
  
  ![snippet_monitor_1](/images/encryption_bypass_part2/8.png)
 
 ### 5. Memory:
 
 This feature of devtools will help you to take heap snapshots, and you can search out the domain content with the sequence of javascipt file gets executed, this feature sometimes make your work easier than following the complex process to find the encryption logic or the file which contains that logic.
 
 We can search the javascript files execution and filter out the files which may contains the logic of encryption.
 
 **Steps to use this:** 
 For the practical purpose we can take example of our vulnerable lab: [https://172.17.0.2/Lab3/login.php](https://172.17.0.2/Lab3/login.php)  
 
 **Note:** If you have not played with this lab, get the docker from [here](https://hub.docker.com/r/bhattsameer/jsdebugginglab) and start playing.  
 
 a.) Access the vulnerable app, we are at OTP.php screen. Open up the devtools and navigate to memory section, click on take snapshot.  
 
 ![memory_1](/images/encryption_bypass_part2/9.png)
 
 b.) Once the process is completed, search for 172.17 and observe the output.  
 
 ![memory_2](/images/encryption_bypass_part2/10.png)
 
 c.) When you scroll down, you will get the method and at right side the file name.  
 
 ![memory_3](/images/encryption_bypass_part2/11.png)
 
 ### 6. Network Initiator:
 
 This one is the menu feature of network section, you can search the method execution sequence in network for each URL. This will help us in finding the logic or method for each page wise, i.e. for OTP.php, otp is getting encrypted when we click on submit button.
 
 **Steps to use this:** 
 For the practical purpose we can take example of our vulnerable lab: [https://172.17.0.2/Lab3/login.php](https://172.17.0.2/Lab3/login.php)
 
 a.) Access the vulnerable app, we are at OTP.php screen. Open up the devtools and navigate to network section. Enter wrong OTP in the application and click on login button. Click on the URL and navigate to "initiator", observe the ***otplogear*** method.  
 
 ![network_1](/images/encryption_bypass_part2/12.png)
 
 ### 7. Resource Saver:
 
 This is a chrome extension, which helps you to download all files which are loaded on client side, once the files are downloaded you can observe them locally through verious tools for finding logics, links in it and so on.
 
 **Steps to use this:**  
 a.) Download and install the extension in chrome using link: [https://github.com/up209d/ResourcesSaverExt](https://github.com/up209d/ResourcesSaverExt)  
 b.) Navigate to target application. Open the devtools, navigate to "Resource Saver" and click on save all resources.  
 
 ![resource_saver_1](/images/encryption_bypass_part2/13.png)
 
 ### 8. Requestly:
 
 This is a chrome extension, which helps you to play with source tab workspace. Means you can replace the application's javascript code with your custom javascript code by redirecting it to localhost.
 
 This will help you when there are multiple changes needs to be perform everytime when the pages refreshes, as with every refresh all your changes went away, even you have to run the snippet code again.
 
 **Steps to use this:**  
 For the practical purpose we can take example of our vulnerable lab: [https://172.17.0.2/Lab3/login.php](https://172.17.0.2/Lab3/login.php)
 
 a.) Access the vulnerable application, we are at OTP.php screen. Open up the devtools and observe ***otplogear*** method in file **ExternalCustom.js***. Observe the code condition "a == 1".  
 
 ![requestly_1](/images/encryption_bypass_part2/14.png)
 
 b.) Create one ***custom.js*** where the change of code is the condition "a == 0". Which means even if we enter wrong OTP, application should redirect us to dashboard page.  
 
 ![requestly_2](/images/encryption_bypass_part2/15.png)
 
 c.) Install the requestly extension from [chrome extension store](https://chrome.google.com/webstore/detail/requestly-redirect-url-mo/mdnleldcmiljblolnjhpnblkcekpdkpa?hl=en-GB) and create one rule as below, here we are redirecting the [http://172.17.0.2/Lab3/js/ExernalCustom.js](http://172.17.0.2/Lab3/js/ExernalCustom.js) to [http://127.0.0.1/JS_TALK/custom.js](http://127.0.0.1/JS_TALK/custom.js)    
 
 ![requestly_3](/images/encryption_bypass_part2/6.png)
 
 d.) Reload the application and observe in devtools, one more workspace added in source tab with 127.0.0.1 and the original ***ExternalCustom.js*** is disappeared.  
 
 ![requestly_4](/images/encryption_bypass_part2/17.png)
 
 e.) Now when you click on Login again with wrong otp, Boom!! you are logged into the application :))

### Conclusion:  

This blog was to help you in making the debugging process easier to find the encryption logic, using some tips and tricks of devtools. The real world of encryption is sure lot more complex than this one but being ready with all the tools and tricks is always helful to be motivated.:)) 
There are lot more stuff about doing debugging with chrome devtools, please have a look at [https://medium.com/frontmen/art-of-debugging-with-chrome-devtools-ab7b5fd8e0b4](https://medium.com/frontmen/art-of-debugging-with-chrome-devtools-ab7b5fd8e0b4)

I enjoyed writing this article and hope that you enjoyed reading it.

In part 3, we will get rid of this continues process, and create our own snippet to fuzz the encrypted parameters using devtools.

Thank you for your time and stay tuned for more!

If you have ways which you have found let me know as well :))  

Follow me:  
 Twitter : [@sameer_bhatt](https://twitter.com/sameer_bhatt5)  
 Github  : [bhattsameer](https://github.com/bhattsameer)
 
 [![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbhattsameer.github.io%2F2021%2F02%2F14%2Fclient-side-encryption-bypass-part-2.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)
   
   ### Reference:
   
   https://blittle.github.io/chrome-dev-tools/  
   https://javascript.info  
   https://developers.google.com/web/tools/chrome-devtools   
   https://medium.com/frontmen/art-of-debugging-with-chrome-devtools-ab7b5fd8e0b4
   
   
  
