# Borland-AccuRev-StackoverFlow
i try to Exploit recently published  vunluerblity in Borland AccuRev 

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
there was tow fake information in ZDI report

1. maping between string and function is "service_Setup_doit" not "service_startup_doit"
2. vunlrabe parameter is debuglog not licfile 

RLM come with AccuRev but it can be directly accsible  by flowing link:  http://www.reprisesoftware.com/license_admin_kits/rlm.v11.3BL1-x86_w1.admin.exe

si i start search service_Setup_doit
