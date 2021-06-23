---
layout: "post"
title: "Intercepting Flutter iOS Application"
date: "2021-06-23 10:20"
excerpt: "Sharing my experience of how I have intercepted the traffic of Flutter based iOS application for dynamic analysis, Also we will see the root detection and SSL verification bypass method I have used."
comments: false
---

### **TL;DR**
Hi, this is [Debugger](https://github.com/bhattsameer) ready to debug Mobile Application.

In this blog I will share how I have intercepted the traffic of Flutter based iOS application for dynamic analysis, Also we will see the root detection and SSL verification bypass method I have used. 

<details><summary>TL;DR Quote</summary>
<p>

```
 Before trying to break any logic,  
 It is always a plus point to understand how that stuff is actually works and implemented.
```  
</p>
</details>

### Introduction  

As Flutter uses dart, Dart is not proxy aware and uses its own certificate store. Hence, The application doesn’t take any proxy settings from the system and sends data directly to server, because of this we cannot intercept the request using [Burpsuite](https://portswigger.net/burp) or any MITM tool, so changing the proxy settings in wifi or trusting any certificate won't help here.  

**Questions:**  
So what we can do in this situation?  

There are multiple approches here to intercept the traffic:  
**Thought 1**:  

SSH into your iOS device and use iptables to redirect the traffic.    
***Note: iptables won't work in iOS device as it requires kernel support.***

**Approche 1**:  
Create a WIFI hotspot using a second WIFI adapter and use iptables to redirect all traffic to Burp.  

**Approche 2**:  
Using OpenVPN to create a tunnel and use iptables to redirect all the traffic to Burp.  

You can read about this all approches in details using [nviso blog](https://blog.nviso.eu/2020/06/12/intercepting-flutter-traffic-on-ios/).  
I have used **Approche 2** to intercept the traffic on the application I am working on.  

### Pre-Requisites:

1. [Jailbroken](https://canijailbreak.com/) iOS device.  
2. [Burp Suite](https://portswigger.net/burp) up and running.  
3. [OpenVPN](https://apps.apple.com/us/app/openvpn-connect/id590379981) application installed in your iOS device.  
4. Liberty - Root detection bypass [Cydia Repo](https://ryleyangus.com/repo).  
5. Filza - To extract ipa from iOS device.
6. Your system and mobile device must be connected to same wifi network.  

### Let's Get Started:  

A. First install OpenVPN application to the iOS device from app store:  

<img src="/images/Intercepting_flutter_iOS_application/1.png" width="200" />

B. Install Your Flutter application to iOS device:  
  As the application has jailbreak detection implemented, Hence when we start application it will crash on splash screen.   
  To bypass the jailbreak detection we can use **Liberty**. You can install Liberty from **[Cydia](https://en.wikipedia.org/wiki/Cydia)** using [Source repo](https://ryleyangus.com/repo)  
  i. Once you have installed Liberty, Go to your device Settings and scroll down a little and you can see the Liberty app in it.  
  ii. Click on Liberty -> Block Jailbreak Detection -> select the desired application of which we wanted to bypass the jailbreak detection.  
 <img src="/images/Intercepting_flutter_iOS_application/2.png" width="200" />


C. Now we can run the app in jailbroken device, let's move ahead.

**Brief Process:**  
 1. We are going to create one OpenVPN connection file, Configure it in our iOS device using OpenVPN application and stablish the connection.  
 2. Using Iptable command we will route the device traffic through our system.  
 3. find and analyze the binary which contains the SSL verification code.  
 4. Using frida we will bypass the SSL verification implementation.

**Let's go step by step with the process:**  
 1. Create OpenVPN file to connect:  
    Use below command to download one script which helps us in creating OpenVPN file as per our need.  
    Script: https://github.com/Nyr/openvpn-install  
    
    ```bash
    > sudo wget https://git.io/vpn -O openvpn-install.sh
    > sudo sed -i "$(($(grep -ni "debian is too old" openvpn-install.sh | cut  -d : -f 1)+1))d" ./openvpn-install.sh
    > sudo chmod +x openvpn-install.sh
    > sudo ./openvpn-install.sh
    ```  
    
    After running Scripts Select Below options:  
    ```
    Which IPv4 address should be used?  
    > Select option of your system IP. i.e. 192.168.1.118
    
    Public IPv4 address / hostname []:
    > Provide your system IP i.e. 192.168.1.118
    
    Protocol [1]: 1
    Port [1194]: 1194
    DNS server [1]: 1
    Name [client]: debugger_test
    ```  
    
    And Press enter. OpenVPN file will be created at **/root/debugger_test.ovpn**

 2. Adding OpenVPN file to device:  
    Run python http server using below command in your system and download the ovpn file to your device.  
    ```bash
    > python3 -m http.server 8081 --directory /root/
    ```
    ![5.png](/images/Intercepting_flutter_iOS_application/5.png)
    
    Open the downloaded ovpn file using OpenVPN, configure and connect to it.  
    ***Note***: You can start openvpn in your system through below command:  
    ```bash
    > sudo service openvpn start
    ```
    <p float="left">
      <img src="/images/Intercepting_flutter_iOS_application/6.png" width="200" />
      <img src="/images/Intercepting_flutter_iOS_application/7.png" width="200" /> 
      <img src="/images/Intercepting_flutter_iOS_application/8.png" width="200" />
    </p>  
    
 3. Route the traffic and burp proxy configuration:  
    Run below commands to route the traffic from your iOS device through your system.  
    ***Note***: In the last command the provided IP is your iOS device IP i.e. 192.168.1.101.  
    ```bash
    > sudo iptables -t nat -A PREROUTING -i tun0 -p tcp --dport 80 -j REDIRECT --to-port 8080
    > sudo iptables -t nat -A PREROUTING -i tun0 -p tcp --dport 443 -j REDIRECT --to-port 8080
    > sudo iptables -t nat -A POSTROUTING -s 192.168.1.101/24 -o eth0 -j MASQUERADE
    ```  
    
    Once all done open burpsuite proxy tab and configure it as below:

    ![9.png](/images/Intercepting_flutter_iOS_application/9.png)
    
    ![10.png](/images/Intercepting_flutter_iOS_application/10.png)  


 4. Identified SSL verification implemented using x509.cc.  
    Run the application and observe the burpsuite dashboard -> Event logs.  
    We got TLS handshake error, Hence we can not intercept https traffic.  

    ![11.png](/images/Intercepting_flutter_iOS_application/11.png)  
    

 5. Use the **Filza** to download the application folder from iOS device.  
    You can install Filza from **Cydia** using [Source repo](https://apt.thebigboss.org/repofiles/cydia/)  
    
    Start the WebDAV server of Filza from settings and Access it through your system browser by accessing http://<Your\_mobile\_IP>:11111.  
    ***Note***: Make sure your phone and system are connected to same wifi network.
    
    In your phone open Filza and navigate to: **/var/containers/Bundle/Application/**  
    Observe the screen and create a zip of your application folder, you can do it by long press on the folder.  
    
    In browser Filza, access to same path and download the zip file.  

    ![3.png](/images/Intercepting_flutter_iOS_application/3.png)  

 6. Extract Zip in your system and open it up through Terminal:  
    
    Binary: /Runner.app/Frameworks/App.framework/App  
    This contains application flutter code.

    Binary: /Runner.app/Frameworks/Flutter.framework/Flutter  
    This contains the x509.cc SSL verification code.

    And to bypass the SSL verification we have to analyze the Flutter binary.  

    ![4.png](/images/Intercepting_flutter_iOS_application/4.png)  

 7. Install Ghidra and dissemble the Flutter binary.  
    Binary path Runner.app/Framworks/Flutter.framework/Flutter  
    Start Ghidra and drag and drop the Flutter binary to it, select all default details.  
    Wait for ghidra to complete the analysis process.  
   
    ![12_0.png](/images/Intercepting_flutter_iOS_application/12_0.png)  

    Once Analysis is completed search for x509.cc string:  
    ![12_1.png](/images/Intercepting_flutter_iOS_application/12_1.png)  
   
    Also perform Scalar search for Magic number 0x186.
    Now to understand what is this magic number you should read this blog by [nviso](https://blog.nviso.eu/2020/05/20/intercepting-flutter-traffic-on-android-x64/) Team.  
    ![13.png](/images/Intercepting_flutter_iOS_application/13.png)  

    Click on the output you got from string search of x509.cc and you will moved to the address directally.  
    
    ![14_0.png](/images/Intercepting_flutter_iOS_application/14_0.png)  

    As we can see, we have multiple XREF[0] against the x509.cc line. We have to identify the valid function address, For that we will take help of our scalar search output which we have done previosly.  
    
    ![14_1.png](/images/Intercepting_flutter_iOS_application/14_1.png)  

    Map scalar search near to the x509.cc output and observe that one address we found in scalar search is in the range of the XREF[0] we got for x509.cc.  

    ![15.png](/images/Intercepting_flutter_iOS_application/15.png)  

    Click on that address range FUN\_0044be60:0044bf94(\*) and Observe the initial bytes value of FUN_0044be60. Copy those bytes:  
   
    ![16.png](/images/Intercepting_flutter_iOS_application/16.png)  

    Copy the inital bytes as below and send to binwalk for offset count if needed.  
    
    ![17.png](/images/Intercepting_flutter_iOS_application/17.png)  
    
    Replace the address value in the below frida script as a pattern variable.
   
    ```js
    function hook_ssl_verify_result(address)
    {
        Interceptor.attach(address, {
        onEnter: function(args) {
            console.log("Disabling SSL validation")
        },
        onLeave: function(retval)
        {
            retval.replace(0x1);
        }
        });
     }
    function disablePinning()
    {
       var pattern = "ff 03 05 d1 fc 6f 0f a9 f8 5f 10 a9 f6 57 11 a9 f4 4f 12 a9 fd 7b 13 a9 fd c3 04 91 08 0a 80 52"
       Process.enumerateRangesSync('r-x').filter(function (m) 
       {
          if (m.file) return m.file.path.indexOf('Flutter') > -1;
          return false;
       }).forEach(function (r) 
       {
          Memory.scanSync(r.base, r.size, pattern).forEach(function (match) {
          console.log('[+] ssl_verify_result found at: ' + match.address.toString());
          hook_ssl_verify_result(match.address);
         });
       });
     }
    setTimeout(disablePinning, 1000)
    ```  

    Connect mobile using USB with your system and run ***frida-ps -Uai***  
    
    ![19.png](/images/Intercepting_flutter_iOS_application/19.png)  
    
    Run frida script to your application package Name using below command:
    ```bash
    > frida –Uf `package` -l `Bypass script` --no-pause
    ```  
    ![20.png](/images/Intercepting_flutter_iOS_application/20.png)  
    
    Perform some activity in the application and check back to burp to see the requests from app being intercepted now:  

    ![21.png](/images/Intercepting_flutter_iOS_application/21.png)  
    ![22.png](/images/Intercepting_flutter_iOS_application/22.png)  

### Conclusion:  
   This blog was to share how I have bypassed the security implementation of an iOS application, and how I have intercepted the traffic of flutter iOS application. As the method for the same is different compare to what we actually do in mobile application testing to intercept the traffic.
    
   I enjoyed writing this article and hope you find this useful.
    
   Thank you for your time and stay tuned for more!
   
You can find me here:  
  Twitter : [@sameer_bhatt5](https://twitter.com/sameer_bhatt5)    
  Github  : [bhattsameer](https://github.com/bhattsameer)  
  LinkedIn: [bhatt-sameer](https://linkedin.com/in/bhatt-sameer)  
  
  <a href="https://www.buymeacoffee.com/bhattsameer" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Tea" height="35" width="154"></a>

### Reference:  
   Huge Thanks to Nviso Team.  
   https://blog.nviso.eu/2020/06/12/intercepting-flutter-traffic-on-ios/
