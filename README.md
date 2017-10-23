# Windows Research Kernel Hacking

## Introduction

As a programmer, if you stay in the game long enough, it is highly likely that you will eventually begin to ask yourself what really makes a computer tick.  If you remove the processes involved in fabricating hardware and confine yourself purely to software, we can agree that the elusive kernel is the place where all the magic happens, even though many have never actually seen it.  While the concepts of the kernel may be understood by many programmers from a theoretical perspective, it is rare to encounter an individual who has compiled one, even more rare to find someone who has successfully made changes to one, and almost non-existent when you consider these concepts in the perspective of the Windows Kernel (as opposed to say Linux).

This makes sense as the Windows Kernel has historically been closed-source.  You couldn't see inside of it if you wanted to.

 Except, this isn't entirely true…

Way back in the 2000's Microsoft released source code for something known as the "Windows Research Kernel" or WRK, here is a portion of the release notes:

"The WRK packages core Microsoft Windows XP x64/Server 2003 SP1 kernel source code with an environment for building and testing experimental versions of the Windows kernel for use in teaching and research.  The WRK includes source for processes, threads, LPC, virtual memory, scheduler, object manager, I/O manager, synchronization, worker threads, kernel heap manager, and other core NTOS functionality."

 This code was offered to educators under a license that mentions that following:

"Use of the Windows Research Kernel requires academic affiliation with an accredited institution of higher education and direct involvement in teaching and/or research"

Per it's intent, [the WRK has been used in a variety of institutional course](https://www.microsoft.com/resources/sharedsource/windowsacademic/facultyexperiences/hpi.mspx)s focused on Operating System research, design, and development.   As a result, there exists materials (ableit dated), that can provide us with a mechanism to glean knowledge of Operating Systems in general.  Even more exciting is that this source code has probably seen very few eyes when compared to other kernel sources. You are truly in for a unique and intimate experience with the Windows Research Kernel.

## Compiling WRK on Windows 10

**Prerequisites:**

- Windows 10
- Visual Studio 2017
- [Windows Server 2003 Evaluation Edition](https://www.microsoft.com/en-us/download/details.aspx?id=19727)
- [Windows Research Kernel](http://gate.upm.ro/os/LABs/Windows_OS_Internals_Curriculum_Resource_Kit-ACADEMIC/WindowsResearchKernel-WRK/)
- [msvcp71.dll](https://www.dll-files.com/msvcp71.dll.html)
- [msvcr71.dll](https://www.dll-files.com/msvcr71.dll.html)

**Copy required dlls**

*Copy msvcp71.dll & msvcr71.dll to C:\Windows\SysWow64 on the host machine.  These libraries are required to successfully compile the Windows Research Kernel on Windows 10.  

**WARNING : If you do not copy the folder above, compilation will appear successful but will inappropriately link.  This will result in issues later on when we load into Server 2003**

**Compling via Command-Line**

Begin by navigating to the WRK-v1.2 folder in a command prompt and execute Build.bat

When executing without any parameters, this will kick off an x86 build of WRK

![CompileViaCommand](/images/CompileViaCommand.png)

**Compling using VS 2017**

Navigate to and open the WRK.sln

![CompileVSsln](/images/CompileVSsln.png)

Select "Ok" to initiate the one-way project upgrade when prompted:

![UpgradePrompt](/images/UpgradePrompt.png)

You _may_ be prompted to install missing features, if so, you should probably install them:

![FeaturesPrompt](/images/FeaturesPrompt.png)

If you receive the following message, click the link area to install the tools for [Windows desktop development with C++ in Visual Studio](https://blogs.msdn.microsoft.com/vcblog/2017/04/17/windows-desktop-development-with-c-in-visual-studio/)

![MigratePrompt](/images/MigratePrompt.png)

Open the solution again, if you are prompted to migrate again, go ahead:

![OpenInVS](/images/OpenInVS.png)

Set the build configuration to x86 and Build, you should see success:

![BuildVS](/images/BuildVS.png)

We are now ready to make some changes to the source code =)

## Modifying the WRK

Expanding the WRK/base/ntos directory reveals the following structure:

    cache\  - cache manager

    config\ - registry implementation

    dbgk\   - user-mode debugger support

    ex\     - executive functions (kernel heap, synchronization, time)

    fsrtl\  - file system run-time support

    io\     - I/O manager

    ke\     - scheduler, CPU management, low-level synchronization

    lpc\    - local procedure call implementation

    mm\     - virtual memory manager

    ob\     - kernel object manager

    ps\     - process/thread support

    se\     - security functions

    wmi\    - Windows Management Instrumentation

    inc\    - NTOS-only include files

    rtl\    - kernel run-time support

    init\   - kernel startup

We will begin by modifying a syscall within _WRK-v1.2/base/ntos/ex/sysinfo.c_

Head to line 1721 (Ctrl+G _then_ :1721) then add the following:

_static int NumTimesCalled = 0;_

![ModifyVS](/images/ModifyVS.png)

Shortly after this (immediately before the line "Status = STATUS\_SUCCESS"), add the following line:

DbgPrint("WRK  %d: Entering NTQuerySystemInformation!<YOURNAME>\n",++NumTimesCalled);

Be sure to susbstitute <YOURNAME> with your name to leave your legacy in the kernel!

Next, go ahead a recompile the WRK using the x86 configuration

![AddNameVS](/images/AddNameVS.png)

## Setting up Windows Server 2003 in Hyper-V

Prerequisites:

  [Enable Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)

Open Hyper-V Manager and create a new Virtual Machine

![HyperV](/images/HyperV.png)

Name the VM, then **be sure to specify as Generation 1** , next you can set the memory to whatever size you prefer and skip the "Configure Networking Step"

![CreateVM](/images/CreateVM.png)

In the "Connect Virtual Disk Step", select "Use an existing virtual hard disk" and point it to the path which contains "Win2k3R2EE.vhd"

![AttachDisk](/images/AttachDisk.png)

Select "Finish" to begin deployment of the Virtual Machine

![FinishVM](/images/FinishVM.png)

Connect to the newly deployed VM

![ConnectVM](/images/ConnectVM.png)

Some useful commands:

_Ctrl + Alt + Left = relinquish mouse capture_

_Ctrl + Alt + End = Send Ctrl + Alt + Delete to the VM_

Go through the initial setup, then login with

_Username = Administrator_

_Password = Evaluation1_

![LoginVM](/images/LoginVM.png)

## Configuring Server 2003 to boot your modified kernel

Prerequisites:

 A way to create an .iso, we will use [ImgBurn](http://www.imgburn.com/)

#### **Install Imgburn with defaults** ####

Use the installer included in the tools folder or acquire a copy online

#### **Package necessary files to an ISO** ####

Open ImgBurn and select "Create image file from files/folder"

![PackageISO](/images/PackageISO.png)

**Add the following files** :

(Your compliled kernel)

_… \OperatingSystems\Windows\_OS\_Internals\_Curriculum\_Resource\_Kit-ACADEMIC\WindowsResearchKernel-WRK\WRK-v1.2\base\ntos\BUILD\EXE\wrkx86.exe_

(The pre-compiled Hardware Abstraction Layer)

_… \OperatingSystems\Windows\_OS\_Internals\_Curriculum\_Resource\_Kit-ACADEMIC\WindowsResearchKernel-WRK\WRK-v1.2\WS03SP1HALS\x86\halacpim\halacpim.dll_

(The kernel boot command for the boot.ini bootstrapper)

… \OperatingSystems\Toolz\KernelBootCommand.txt

(The entire folder for DebugView\_v4.81 which includes the Windows Debugging Tools) - **Use the Add Folder option**

… \OperatingSystems\Toolz\DebugView\_v4.81

**Build the .iso**

![BuildISO](/images/BuildISO.png)

**Load your files into the VM**

Load your newly minted .iso into Server 2003 by selecting "Media => DVD Drive => Insert Disc"

You should see your files appear in the OS

![LoadISO](/images/LoadISO.png)

**Copy Kernel & HAL to System 32 on the VM**

Copy your wrk86.ese and halacpim.dll files to C:\Windows\System32 on the virtual machine

**Edit the Boot.ini file**

To view and edit the Boot.ini file, follow these steps:

1. Click **Start** , point to
**Settings** , and then click **Control Panel**.
2. In Control Panel, double-click
**System**.
3. Click the **Advanced** tab, and then click
**Settings** under **Startup and Recovery**.
4. Under **System startup** , click
**Edit**.
5. Copy the content from **KernelBootCommand.txt**  and paste it at the very bottom of Boot.ini

![EditBootIni](/images/EditBootIni.png)

1. Ensure that you have saved the modified Boot.ini
2. Poweroff or reboot the Virtual Machine

#### **Debugging your modified Kernel**

On reboot, you should see a new option to boot the WRK kernel with debugger enabled, select that option and proceed as normal

![BootWRK](/images/BootWRK.png)

Open Dbgview.exe and select "Capture => Capture Kernel"

![OpenDBGView](/images/OpenDBGView.png)

Notice that your customized syscall is printed to the debugger each time it is called

![CaptureSyscall](/images/CaptureSyscall.png)

#### **Questions**

1. Why is the NTQuerySystemInformation syscall called so often?
2. What does the NTQuerySystemInformation syscall do?
3. How has your understanding of kernels changed as a result of this exercise?
4. Can you describe how the Windows Subsystem for Linux allows for execution of unmodified Linux binaries on Windows 10?
5. Can you describe how Docker's isolation of multiple userspaces across a shared kernel allows for portability of containers?

#### **Extra Challenges**

1. Modify and track calls to a different syscall within the WRK
2. Create your own syscall – see: [https://www.dcl.hpi.uni-potsdam.de/research/WRK/2009/03/implementation-of-a-new-system-service-call-2009-update/](https://www.dcl.hpi.uni-potsdam.de/research/WRK/2009/03/implementation-of-a-new-system-service-call-2009-update/)
3. Demonstrate porting a Linux syscall to an NT syscall a la the Windows Subsystem for Linux – see: [https://blogs.msdn.microsoft.com/wsl/2016/06/08/wsl-system-calls/](https://blogs.msdn.microsoft.com/wsl/2016/06/08/wsl-system-calls/)
