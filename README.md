# f5-waf-tester
F5 Networks Advanced WAF tester tool to ensure basic security level.

# Overview

F5 Networks Threat Research Team has created a tool that provides an easy and fast way to integrate security testing as part of the SDLC process for basic application protection health check before moving to production.

The tester tool covers testing of varoius basic attack types which include:

        "Cross Site Scripting (XSS)"    
        "SQL-Injection"    
        "NoSQL Injection"    
        "Command Execution"    
        "Path Traversal"    
        "Predictable Resource Location"    
        "Detection Evasion"    
        "Insecure Deserialization"    
        "HTTP Parser Attack"    
        "XML External Entities (XXE)"    
        "Server-Side Request Forgery"    
        "Server Side Code Injection"    


# How it Works

The tool is intended to test the WAF configuration state and its provided security posture against common web attack types. The tool will send HTTP requests containing attacks and will expect to recieve a WAF blocking page in the resposnse. In case the attack vecor was not blocked, the tool will read the WAF logs and its confgiuration to try determine possible reasons for the attack not being blocked, and suggest corresponding actions.

On top of the above attack type tests, the tool supports focusing on testing spesific server technologies:

        "Node.js"
        "PHP"
        "MongoDb"
        "Micorsoft Windows"
        "Unix/Linux"
        "XML"
        "Java Servlets/JSP"
        "ASP.NET"
        "Python"

# Installation

## Prerequisites

Python 2.7+\
Python package control (pip):\
Ubuntu/Kali, ```sudo apt-get install -y python-pip```  
Fedora, ```sudo dnf install -y python-pip``` 

Install the tool. ```git clone https://github.com/f5devcentral/f5-waf-tester.git```  

# How to Use

#### 1. [Intail Setup] Create configuration file for the first time:  ```f5-waf-tester --init``` 

that will contain initial information about the testing environment which should \ include information the application server technologies.


```
[BIG-IP] Host [1.1.1.1]:                        <<< The BIG-IP Mgmt IP address to be tested
[BIG-IP] Username [username]:                   <<< The BIG-IP Mgmt username to be tested
[BIG-IP] Password [********]:                   <<< The BIG-IP Mgmt password to be tested
ASM Policy Name [policy_name]:                  <<< The WAF policy name to be tested
Virtual Server URL [https://2.2.2.2]:           <<< The protocol and virtual address that will be tested>
Blocking Regular Expression Pattern [<br>Your support ID is: (?P<id>\d+)<br>]:          <<< The blocking response page string to expect from ASM  
Number OF Threads [25]:                 <<< The number of threads to open in parallel
[Filters] Test IDs to include (Separated by ',') []:            <<< You can choose a spesifc test ID`s that will be tested 
[Filters] Test Systems to include (Separated by ',') [Unix/Linux,Node.js,MongoDb,Java Servlets/JSP]:            <<< You can choose a spesifc systems names that will be tested 
[Filters] Test Attack Types to include (Separated by ',') []:           <<< You can choose a spesifc attack types names that will be tested
[Filters] Test IDs to exclude (Separated by ',') [,]:           <<< You can choose a spesifc test ID`s not that will be tested (on top of the include list)
[Filters] Test Systems to exclude (Separated by ',') []:                <<<  You can choose a spesifc system names not that will be tested (on top of the include list)
[Filters] Test Attack Types to exclude (Separated by ',') [],]:                 <<< You can choose a spesifc attack type names not that will be tested (on top of the include list)
```

After the first init, config file (config.json) will be created by default under python folder.
The default can be changed using "-c" with the desired path, as well "-t" for the test file and "-r" for the report file.


More information can be observed by clicking ```f5-waf-tester --help```
```
usage: f5-waf-tester [-h] [-v] [-i] [-c CONFIG] [-t TESTS] [-r REPORT]

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -i, --init            Initialize Configuration. (default: False)
  -c CONFIG, --config CONFIG
                        Configuration File Path. (default:
                        /usr/local/lib/python2.7/dist-
                        packages/awaf_policy_validator/config/config.json)
  -t TESTS, --tests TESTS
                        Tests File Path. (default: /usr/local/lib/python2.7
                        /dist-
                        packages/awaf_policy_validator/config/tests.json)
  -r REPORT, --report REPORT
                        Report File Save Path. (default: report.json)
  ```

#### 2. [Deploy] Run the tester tool and observe the results: ```f5-waf-tester```

The results of the tests will be saved to "report.json" under the current folder. 
Test results summary provide total number of passed/failed tests:

e.g:

```
 "summary": {
    "fail": 4,
    "pass": 45
  }
  ```
    
  fail - The attack was not block by the WAF\
  pass - The attack was bloacked by the WAF \
  
  On top of the summart results, the tester tool provide infromation for each test.
  
  e.g: 
  
  ```
     "100000025": {                                     <<< Test ID
      "CVE": "",
      "attack_type": "Server Side Code Injection",  
      "name": "ASP.NET Code Injection",
      "results": {
        "header": {                                      
          "expected_result": {
            "type": "signature",                        <<< Attack Type
            "value": "200004516"                        <<< Signture ID in WAF
          },
          "pass": false,
          "reason": "Attack Signature is not in the ASM Policy",
          "support_id": ""
```
  
  If the test ID passed, the "support_id" field provide the corresponding support ID that was blocked by the WAF \
  If the test ID fail -  the "reason" field descibe the possible reason why the WAF did not block the request \
  if the test ID vector is signature - the "value" field provide the signature ID that related to this attack in WAF
  
  Here are the possible reasons that could cause the test ID to fail:
  ```
  ASM Policy is not in blocking mode
  Attack Signature is not in the ASM Policy
  Attack Signatures are not up to date
  Attack Signature disabled
  Attack Signature is in staging
  Parameter * is in staging
  URL * is in staging
  URL * Does not check signatures
  Header * Does not check signatures
  Evasion disabled
  Evasion technique is not in blocking mode
  Violation disabled
 ```
  

The testing results can be found on the same path under "report.json" file.\
The configuration and testing files can be edited based on the testing results to describe exactly the application environments.\

If needed, edit the config file ("config.json") to exclude or include tests based on the tests results:
e.g: Include only the server technologies that related to the application strcutre:

e.g:
```
    "include": {
      "attack_type": [],
      "id": [],
      "system": [
        "Unix/Linux",
        "Node.js",
        "MongoDb",
        "Java Servlets/JSP"
      ]
 }
 ```

#### 3. [Inspect and Adapt] Refine the WAF policy based on the possilbe reasons results and run the tester tool again
Once the results are clear from any failed tests, it means that you WAF is ready to be deployed and provide the basic protection level against web attacks.

#### 4. [Add Attacks] Build your custom tests to cover more use cases 
open the "tests.json" file and add your own test vector blocks based on the same JSON format.  
Follow the tests samples in the tests.json file as an examples to understand how to build a new attacks.

#### Disclaimer
The tool is not testing whether the application itself is vulnerable and also tests only a subset of attacks. The tool tries to test the WAF security policy level, and is not a replacement for a vulnerability scanner assessment.
