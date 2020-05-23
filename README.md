# boko Application Hijack Scanner for macOS
boko.py is a static application scanner for macOS that searches for and identifies potential dylib hijacking and 
weak dylib vulnerabilities for application executables, as well as scripts an application may use that 
have the potential to be backdoored. The tool also calls out interesting files and lists them instead of manually 
browsing the file system for analysis.  

The reason behind creating this tool was because I wanted more control over the data Dylib
Hijack Scanner discovered. The original scanner also seems to stop once it discovers the first case of a vulnerable Dylib without expanding the rest of the rpaths. Since sometimes the first result is expanded in a non-existent file within a SIP-protected area, I wanted to get the rest of those expanded paths. Because of this, there are false positives, so the tool assigns a certainty field for each item.  
| **Certainty**            |  **Description** |
|--------------------:|:-----------------------------------|
| Definite        | The vulnerability is 100% exploitable  |
| High        | If the vulnerability is related to a main executable and rpath is 2nd in the load order, there is a good chance the vulnerability is exploitable  |
| Potential        | This is assigned to dylibs and backdoorable scripts, worth looking into but may not be exploitable  |
| Low        | Low chance this is exploitable because of late load order, but knowledge is power  |

The backbone of this tool is based off of scan.py from [DylibHijack](https://github.com/synack/DylibHijack) by Patrick Wardle (@synack).  

[Check out the Wiki for more information on how to use boko and how to use it as a class for your own purposes.](https://github.com/bashexplode/boko/wiki)

#### Usage:
```Python
boko.py [-h] (-r | -i | -p /path/to/app) (-A | -P | -b) [-oS outputfile | -oC outputfile | -oA outputfile] [-s] [-v]
```

#### Parameters:  
| **Argument**            |  **Description** |
|--------------------:|:-----------------------------------|
| -h, --help          | Show this help message and exit  |
| -r, --running       | Check currently running processes |
| -i, --installed     | Check all installed applications  |
| -p /file.app        | Check a specific application i.e. /Application/Safari.app  |
| -A, --active     | Executes executable binaries discovered to actively identify hijackable dylibs  |
| -P, --passive     | Performs checks only by viewing file headers (Default) |
| -b, --both     | Performs both methods of vulnerability testing  |
| -oS outputfile  | Outputs standard output to a .log file |
| -oC outputfile  | Outputs results to a .csv file |
| -oA outputfile  | Outputs results to a .csv file and standard log  |
| -s, --sipdisabled   | Use if SIP is disabled on the system to search typically read-only paths|
| -v, --verbose       | Output all results in verbose mode while script runs, without this only Definite certainty vulnerabilities are displayed to the console |

It is recommended only to use active mode with the -p flag and selecting a specific program. Also, it's a good idea to use -v with -oS or -oA, unless you are only looking for definite certainty vulnerabilities.

It is highly discouraged to run this tool with the -i and (-A or -b) flags together. This will open every executable on your system for 3 seconds at a time. I do not take any responsibility for your system crashing or slowing down because you ran that. Additionally, if you have dormant malware on your system, this will execute it. 

#### Requirements:

* Python 3  
* `python -m pip install psutil`

#### Process Flow:

##### Passive mode:

###### Running:
* Identify all running processes on system
* Obtain full path of running executable
* Open executables and identify macho headers
* Identify dylib relative paths that are loaded and check if files exist in that location
* Output hijackable dylibs and weak dylibs for running applications

###### Installed/Application:
* Scan full directory of application for all files
* Identify executable files, scripts, and other interesting files in application directory
* Open executables and identify macho headers or if the file is a script
* Identify dylib relative paths that are loaded and check if files exist in that location
* Output hijackable dylibs, weak dylibs, backdoorable scripts, and interesting files (verbose only)

##### Active mode:

###### Running:
* Identify all running processes on system
* Obtain full path of running executable
* Open executables and identify macho headers
* Execute the executables and analyze rpaths that are attempted to load
* Output hijackable dylibs and weak dylibs for running applications

###### Application:
* Scan full directory of application for all files
* Identify executable files, scripts, and other interesting files in application directory
* Open executables and identify macho headers or if the file is a script
* Execute the executables and analyze rpaths that are attempted to load
* Output hijackable dylibs, weak dylibs, backdoorable scripts, and interesting files (verbose only)


#### Suggested Improvements:

* Multi-threading for quicker full system scan

## Coded by Jesse Nebling (@bashexplode)
