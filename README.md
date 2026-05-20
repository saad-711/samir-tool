
# Malware
The word malware is derived from the term MALicious softWARE. Therefore, any software that has a malicious purpose can be considered malware. Malware is further classified into different categories based on its behaviour. However, we will not investigate the details of those in this room. Here, we will ponder the steps we will take if we suspect that we have found malware in a machine. So, let's get started.

# The purpose behind Malware Analysis

Malware analysis is an important skill. As a quick overview, Malware Analysis is performed by the following people in the Security Industry:


## 

 - [Security Operations teams analyse malware to write detections for malicious activity in their networks.](https://awesomeopensource.com/project/elangosundar/awesome-README-templates)
 - [Incident Response teams analyse malware to determine what damage has been done to an environment and to remediate and revert that damage.](https://github.com/matiassingers/awesome-readme)
 - [Threat Hunt teams analyse malware to identify IOCs, which they use to hunt for malware in a network.](https://bulldogjob.com/news/449-how-to-write-a-good-readme-for-your-github-project)
 - [Malware Researchers in security product vendor teams analyse malware to add detections for it in their security products.](https://bulldogjob.com/news/449-how-to-write-a-good-readme-for-your-github-project)
 - [Threat Research teams in OS Vendors like Microsoft and Google analyse malware to discover the vulnerabilities exploited and add more security features to the OS/applications.](https://bulldogjob.com/news/449-how-to-write-a-good-readme-for-your-github-project)

Overall, it seems like many different people do malware Analysis for many compelling reasons. So let's see how to start!
# Before we begin!
Malware can be as dangerous as a weapon if handled carelessly. Before starting any analysis, it’s important to understand how to work safely. Always follow these precautions to protect your system and maintain a controlled environment during your research.: 

-Never analyse malware or suspected malware on a machine that does not have the sole purpose of analysing malware.

-When not analysing or moving malware samples around to different locations, always keep them in password-protected zip/rar or other archives so that we can avoid accidental detonation.

-Only extract the malware from this password-protected archive inside the isolated environment, and only when analysing it.

-Create an isolated VM specifically for malware analysis, which can be reverted to a clean slate once you are done.

-Ensure that all internet connections are closed or at least monitored.

-Once you are done with malware analysis, revert the VM to its clean slate for the next malware analysis session to avoid residue from a previous malware execution corrupting the next one.

# Techniques of malware analysis
Malware Analysis is like solving a puzzle. Different tools and techniques are used to find the pieces of this puzzle, and joining those pieces gives us the complete picture of what the malware is trying to do. Most of the time, you will have an executable file (also called a binary or a PE file. PE stands for Portable Executable), a malicious document file, or a Network Packet Capture (pcap). The Portable Executable is the most prevalent type of file analysed while performing Malware Analysis.

To find the different puzzle pieces, you will often use various tools, tricks, and shortcuts. These techniques can be grouped into the following two categories:

Static Analysis/
Dynamic Analysis

## Static Analysis
When malware is analysed without being executed, it is called Static Analysis. In this case, the different properties of the PE file are analysed without running it. Similarly, in the case of a malicious document, exploring the document's properties without analysing it will be considered Static Analysis. Examples of static analysis include checking for strings in malware, checking the PE header for information related to different sections, or looking at the code using a disassembler. We will look at some of these techniques later .

Malware often uses techniques to avoid static analysis. Some of these techniques use obfuscation, packing, or other means of hiding their properties. To circumvent these techniques, we often use dynamic analysis.

## Dynamic Analysis
Malware faces a dilemma. It has to execute to fulfil its purpose, and no matter how much obfuscation is added to the code, it becomes an easy target for detection once it runs.Image depicting a malware under observation with a microscope

Static analysis might provide us with crucial information regarding malware, but sometimes that is not enough. We might need to run the malware in a controlled environment to observe what it does in these cases. Malware can often hide its properties to thwart Static Analysis. However, in most of those cases, Dynamic Analysis can prove fruitful. Dynamic analysis techniques include running the malware in a VM, either in a manual fashion with tools installed to monitor the malware's activity or in the form of sandboxes that perform this task automatically. We will learn about some of these techniques later in this room. Once we run the malware in a controlled environment, we can use our knowledge from the Windows Forensics rooms to identify what it did in our environment. The advantage here is that since we control the environment, we can configure it to avoid noise, like activity from a legitimate user or Windows Services. Thus, everything we observe in such an environment points to malware activity, making it easier to identify what the malware did in this scenario.

Malware, however, often uses techniques to prevent an analyst from performing dynamic analysis. Since most dynamic analyses are performed in a controlled environment, most methods to bypass dynamic analysis include detecting the environment in which they are being run. Therefore, in these cases, the malware uses a different, benign code path if it identifies that it is being run in a controlled environment.


## Advanced Malware Analysis
Advanced malware analysis techniques are used to analyse malware that evades basic static and dynamic analysis. Disassemblers and debuggers are used to perform advanced malware analysis. Disassemblers convert the malware's code from binary to assembly so that an analyst can look at the instructions of the malware statically. Debuggers attach to a program and allow the analyst to monitor the instructions in malware while it is running. A debugger allows the analyst to stop and run the malware at different points to identify interesting pieces of information while also providing an overview of the memory and CPU of the system.

# Basic static Analysis 
When analysing a new piece of malware, the first step is usually performing basic static analysis. Basic static analysis can be considered as sizing up the malware, trying to find its properties before diving deep into analysis. It provides us with an overview of what we are dealing with. Sometimes, it might give us some critical information, such as what Windows API calls the malware is making or whether it's packed or not. However, other times, it might only give us information to help us size the malware up and give us an idea of the effort required to analyse it.

So without further ado, let's see some of the techniques we can use to perform basic static analysis.

Caution!!!!!!!!!!!!!!!!!!!

Although static analysis is performed without running the malware, it is highly recommended that you perform malware analysis in an isolated Virtual Machine. You can create a clean snapshot of your Virtual Machine before performing any malware analysis and revert it to start from a clean state again after every analysis. Don't perform malware analysis on a live machine that is not purpose-built for malware analysis. For this room, we will be using the attached Remnux VM(opens in new tab). Remnux (Reverse Engineering Malware Linux) is a Linux distribution purpose-built for malware analysis. It has many tools required for malware analysis that are already installed on it.

Examining the file type
Though the file type of malware is often visible in the file extension and is obvious, sometimes malware authors try to trick users by using misleading file extensions. In such scenarios, it is helpful to know how to find the actual file type of a file without depending on file extensions. In Linux, we can find the file type of a file using the file command. To understand what the file command does, we can read its man page or use the --help option:

man file or file --help

We will find out that it is a simple command to use. We can use the following command to find the file type of a file:

file <"filename">


![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_001745599.png)

There is a folder named Samples on the Desktop in the attached VM. We will be using the samples present in that folder for our analysis. The above terminal shows the file command being run on the 'WannaCry' sample. The output shows a PE32(opens in new tab) executable file with a Graphical User Interface, which was compiled for a system that runs Microsoft Windows with an Intel 80386-based processor. The Intel 80386 processor was one of the first 32-bit processors ever, and the instruction set designed for the 80386 is still used for 32-bit Intel processors, which is why you see "x86" processors and code. This means that the "80386" in the output above tells us that this application was designed for 32-bit Intel processors.

## Examining Strings

Another really important command that provides us with useful information about a file is the strings command. This command lists the strings present in a file. To understand what the string command does, we can read its man page or use the --help option:

man strings or strings --help

We will find that it is also a simple command to use. We can use the following command to find the strings in a file:

strings <"filename">

Looking at strings in a file can often give clues related to the behaviour of malware. For example, if we see URLDownloadToFile (opens in new tab)in the output of the strings command, we will know that this malware is doing something with the URLDownloadToFile Windows API(opens in new tab). Most likely, it is downloading a file from the internet and saving it on the disk. Similarly, strings might also provide contextual information that helps us later during malware analysis.

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_001800536.png)

Here we can see the strings command being run against the 'Wannacry' sample. We will see that the output starts with the DOS Stub, which is the text that says !This program cannot be run in DOS mode. Some values don't make much sense and look like garbage, but you will also see useful output. For example, we can see above that some strings look like Windows APIs. For example, CloseHandle, GetExitCodeProcess, TerminateProcess, and so on. Similarly, we can see text that says inflate 1.1.3 Copyright 1995-1998 Mark Adler. A quick search shows that it is a part of the zlib data compression (opens in new tab)library; this tells us that the sample might be using this library.


## Calculating Hashes 

File Hashing provides us with a fixed-size unique number that identifies a file. A File Hash can therefore be considered a unique identifier for a file, similar to Social Security Numbers or National Identification Numbers used for the citizens of a country. Hashing is an important concept in malware analysis. It can be used as an identifier for specific malware. As we will see later in this task, this identifier can then be shared with other analysts or searched online for information-sharing purposes. Please note that a single bit of difference in two files will result in different hashes, so changing the hash of a file is as simple as changing one bit in it.

Commonly, md5sum, sha1sum and sha256sum hashes are used for file hashing. We can calculate file hashes by using a simple command in Linux, as shown below for the md5sum hash:

md5sum <"filename">

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_001814410.png)

Above, we can see the md5sum hash is calculated for the file named WannaCry.

Similarly, sha1sum and sha256sum commands can be used for calculating sha1sum or sha256sum of a file (Hashes are often referred to without the `sum` at the end, e.g. md5 instead of md5sum and so on.)

# AV scans and VirusTotal
Scanning a file using AVs or searching for a hash on VirusTotal(opens in new tab) can also provide useful information about the classification of malware performed by security researchers. However, when using an online scanner, it is recommended to search for the malware's hash instead of uploading it online to avoid leaking sensitive information. Only upload a sample if you are sure of what you are doing.

Let's see what it says about the sample for which we calculated the hash above. We can search for the md5sum we calculated for the WannaCry sample on the VirusTotal homepage:

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_001943151.png)

VirusTotal has a mix of handy features. It provides scan results from 60+ AV vendors and each AV vendor's classification of the sample. 

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_003700919.png)

The details tab lists the history of the sample, the first submission, the last submission, and the metadata of the sample.

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_001955770.png)

Sometimes it also provides information about the behaviour of a sample and its relations as seen in different environments online. 

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002003064.png)

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002010739.png)

We can also find comments about the sample by the community on VirusTotal, which can sometimes provide additional context about the sample.

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002019580.png)

Perhaps it is very clear from the above screenshots that we are looking at a sample of WannaCry ransomware.

# PE Header

The PE File Header(opens in new tab) contains the metadata about a Portable Executable file. This data can help us find a lot of helpful information to help us in our analysis. some of the vital information found in the PE header is explained below:

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002029784.png)

Imports/Exports : 

A PE file seldom contains all the code that it needs to run on a system on its own. Most of the time, it reuses code provided by the Operating System. This is done to use less space and leverage the framework that the Operating System has laid out to perform tasks instead of re-inventing the wheel. Imports are such functions that the PE file imports from outside to perform different tasks.

For example, if a developer wants to query a Windows Registry value, they will import the RegQueryValue(opens in new tab) function provided by Microsoft instead of writing the code themselves. It is understood that this function will be present on any Windows machine on which the developer's code is going to run, so it does not need to be included in the PE file itself. Similarly, any PE file export functions are exposed to other binaries that can use that function instead of implementing it themselves. Exports are generally associated with Dynamically-Linked libraries (DLL files), and it is not typical for a non-DLL PE file to have a lot of exports.

Since most PE files use the Windows API to perform the bulk of their jobs, a PE file's imports provide us with crucial information on what that PE file will do. It becomes evident that a PE file that imports the InternetOpen function will communicate with the internet, a URLDownloadToFile function shows that a PE file will download something from the internet, and so on. Names of Windows APIs are generally intuitive and self-explanatory. However, we can always consult Microsoft Documentation(opens in new tab) to verify the purpose of a particular Windows function.

Sections:

Another useful piece of information available in the PE file header is the information about sections in the PE file. A PE file is divided into different sections which have different purposes. Although the sections in a PE file depend on the compiler or packer used to compile or pack the binary, the following are the most commonly seen sections in a PE file.

.text: This Section generally contains the CPU instructions executed when the PE file is run. This section is marked as executable.

.data: This Section contains the global variables and other global data used by the PE file.

.rsrc: This section contains resources that are used by the PE file, such as images, icons, etc.

## Analysing PE header using the pecheck utility

We can use the pecheck utility present in the Remnux VM attached to the room to check the PE header. 

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_004339453.png)

Here we can see information that PECheck has extracted from the PE header of the WannaCry sample. We see that the sample has four sections, .text, .rdata, .data and .rsrc and their respective entropy. Similarly, it has also shown us the different hashes of the sample. Pecheck also shows us the functions that a PE file imports. In the above terminal window, we can see the IMAGE_IMPORT_DESCRIPTOR, which shows the functions it imports from the ADVAPI32.dll Linked library. We will see similar descriptors for all the other linked libraries whose functions are imported by the sample.

We can see that pecheck shows us a lot more information than what we discussed in this task; however, discussing all that information is out of the scope of this .We will take what we are looking for from the information we see, namely, the section information and the imports of our samples. 

#Basic Dynamic Analysis

While basic static analysis provides us with useful information about a sample, we often need to perform additional analysis to move further in our analysis procedure. One quick and dirty way to find more clues about a malware's behaviour is by performing basic dynamic analysis. Many of the properties of a malware sample can be hidden when it's not running. However, when we perform dynamic analysis, we can lay these properties bare and learn more about the behaviour of a malware sample.

Caution!!!!!!!!!!!!!!
Dynamic analysis requires running live malware samples, which can be destructive. It is highly recommended that you perform malware analysis on an isolated Virtual Machine. You can create a clean snapshot of your Virtual Machine before performing any malware analysis and revert it to start from a clean state again after every analysis. Don't perform malware analysis on a live machine that is not purpose-built for malware analysis.

## Introduction to SandboxesImage representing a sandbox
Sandbox is a term borrowed from the military. A sandbox is a box of sand, as the name suggests, modelling the terrain where an operation has to take place, in which a military team dry runs their scenarios to identify possible outcomes. In malware analysis, a sandbox is an isolated environment mimicking the actual target environment of a malware, where an analyst runs a sample to learn more about it. Malware analysis sandboxes heavily rely on Virtual Machines, which can take snapshots and revert to a clean state when required.

## Construction of a sandbox
For malware analysis using sandboxes, the following considerations make the malware analysis effective:

-Virtual Machine mimicking the actual target environment of the malware sample

-Ability to take snapshots and revert to a clean state

-OS monitoring software, such as Procmon, ProcExplorer, Regshot, etc.

-Network monitoring software, such as Wireshark, tcpdump, etc.

-Control over the network through a dummy DNS server and web server.

-A mechanism to move analysis logs and malware samples in and out of the Virtual Machine without compromising the host (Be careful with this one. If you have a shared directory with your malware analysis VM that remains accessible when running malware, you might -risk malware affecting all files in your shared directory.

# Open Source Sandboxes
Though it is good to understand what a good sandbox is made of, building a sandbox from scratch is not always necessary. One can always set up Open Source Sandboxes. These sandboxes provide the framework for performing basic dynamic analysis and are also customizable to a significant extent to help those with a more adventurous mindset.

## Cuckoo's Sandbox
Cuckoo's sandbox(opens in new tab) is the most widely known sandbox in the malware analysis community. It was developed as part of a Google Summer of Code project in 2010. It is an open-source project that you will often see deployed in SOC environments and with enthusiasts' home labs. Advantages of Cuckoo's sandbox include huge community support, easy-to-understand documentation, and lots of customisations. You can deploy it on your network and let the community signatures guide you into identifying which files are malicious and which are benign because of the vast corpus of community signatures that comes with it.

Cuckoo's sandbox has been archived, and an update is pending. It also doesn't support Python 3, making it obsolete right now. However, all is not lost because we have alternatives.

## CAPE Sandbox
CAPE Sandbox(opens in new tab) is a little more advanced version of Cuckoo's Sandbox. It supports debugging and memory dumping to support the unpacking of packed malware . Though beginners can use this sandbox, advanced knowledge is required to fully use it. A community version of this sandbox is available online, and it can be used to test-run it before installation. CAPE Sandbox is so far actively developed and supports Python 3.

## Online Sandboxes:
Setting up and maintaining a sandbox can be a time-consuming task. Keeping that in view, online sandboxes can be of great help. Some of the most commonly used online sandboxes are as follows:

-Online Cuckoo Sandbox
-Any.run
-Intezer
-Hybrid Analysis
Though online sandboxes provide a useful utility, it is best not to submit a sample online unless you are sure of what you are doing. A better approach is to search for the sample's hash on the service you are using to see if someone has already submitted it. Let's look at Hybrid Analysis to see what interesting analysis it provides for our sample.

# Analysing samples using Hybrid Analysis
On its homepage, we are greeted with the following screen: 

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002041447.png)

As we mentioned, we will not be submitting a sample. Instead, we will search for the hash of our sample. Therefore, we will search for the md5sum of the WannaCry sample from the attached VM. We will see that it has already been submitted multiple times, and we can choose from the submitted results.

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002048909.png)

Let's open the one submitted on Windows 7 64-bit from among these.

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002055614.png)

We will see the above interface when we click on the sample. We can see a navigation pane on the right that highlights different parts of the report. We can also see that the verdict is malicious, with a threat score of 100/100 and AV detection of 95%. Below that, we see the overview of the sample's behaviour. Below that, we will see the mapping to MITRE ATT&CK techniques(opens in new tab). We will see the following mapping when we click view all details:

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002102646.png)

Below that, we will see some indicators, context information and some static analysis information for the sample. The dynamic analysis part comes below that:

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002111265.png)

This part provides us with a lot of information about the behaviour of the sample when it was run in a sandbox. We can click each process to find more details about it. In the above screenshot, the executions of cmd.exe are of particular interest. We can see that the sample is running script files and deleting backups and volume shadow copies, something often done by ransomware operators to stop the victim from restoring their files from these sources.

Below this section, we will see the network analysis of the sample:

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002121089.png)

Extracted strings and extracted files are also available in the report. These can provide information about the batch scripts we saw in the processes above.

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_002126412.png)

There are also comments from the community at the very end. As we might have seen, using the discussed techniques, we can find many pieces of the puzzle that a malware sample is. However, in some cases, these techniques can prove insufficient to make a decision. Let's move to the next task to determine what scenarios can make it challenging to analyse malware.

# Anti-analysis techniques

While the security researchers are devising techniques and tools to analyse malware, the malware authors are working on rendering these tools and techniques ineffective. We found a great deal of information in the previous tasks about the malware we analysed. However, there are ways malware authors can make our lives difficult. Below are some of the techniques used by malware authors to do the same.

## Packing and Obfuscation
Malware authors often use packing and obfuscation to make an analyst's life difficult. A packer obfuscates, compresses, or encrypts the contents of malware. These techniques make it difficult to analyse malware statically. Specifically, a packed malware will not show important information when running a string search against it. For example, let's run a string search against the file named zmsuz3pinwl in the Samples folder in the attached VM. 

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_001800536.png)

We will notice that this sample contains mainly garbage strings that don't provide much value to us. Let's run pecheck on the sample to see what else we get.

![red final samir tool](https://github.com/saad-711/samir-tool/blob/main/image_2026-05-18_001722482.png)

As suspected, we see that the executable has characteristics typical of a packed executable, as per pecheck. We will notice that there is no .text section in the sample, and other sections have execute permissions, which shows that these sections contain executable instructions or will be populated with executable instructions during execution. We will also see that this sample does not have many imports that might show us its functionality, as we saw with the previous sample. 

For analysis of packed executables, the first step is generally to unpack the sample. 

## Sandbox evasion

As we have seen previously, we can always run a sample in a sandbox to analyse it. In many cases, that might help us analyse samples that evade our basic static analysis techniques. However, malware authors have some tricks up their sleeves that hamper that effort. Some of these techniques are as follows:

-Long sleep calls: Malware authors know that sandboxes run for a limited time. Therefore, they program the malware not to perform any activity for a long time after execution. This is often accomplished through long sleep calls. The purpose of this technique is to time out the sandbox.

-User activity detection: Some malware samples will wait for user activity before performing malicious activity. The premise of this technique is that there will be no user in a sandbox. Therefore, there will be no mouse movement or typing on the keyboard. Advanced malware also detects patterns in mouse movements that are often used in automated sandboxes. This technique is designed to bypass automated sandbox detection.

-Footprinting user activity: Some malware checks for user files or activity, such as whether there are any files in the MS Office history or internet browsing history. If no or little activity is found, the malware will consider the machine as a sandbox and quit. 

-Detecting VMs: Sandboxes run on virtual machines. Virtual machines leave artefacts that can be identified by malware. For example, some drivers installed in VMs being run on VMWare or Virtualbox give away the fact that the machine is a VM. Malware authors often associate VMs with sandboxes and would terminate the malware if a VM is detected.

The above list is not exhaustive, but it gives us an idea of what to expect when analysing malware. 
