Setting Up a Programming Environment via Windows 10 Bash
========================================================

Last modified: Jun 10, 2020

Contents:

[1 Get the Windows 10 Subsystem for Linux](#get-the-windows-10-subsystem-for-linux)

  [1.1 sudo apt-get](#sudo-apt-get)

[2 Installing X](#2-x)

[3 Installing the Compilers](#installing-the-compilers)

[4 Installing Eclipse](#installing-eclipse)

[5 Optional packages](#optional-packages)



In 2016, Microsoft added to Windows 10 (64 bit) the ability to run Ubuntu Linux in parallel with Windows. Variously referred to as “Bash on Windows”, or “Ubuntu on Windows”, and more officially as the “Windows Subsystem for Linux” (WSL), this seems to be a very useful way to work with Linux-based software development tools from a Windows 10 machine. Microsoft has announced plans to support other Linux distributions as well.

What it is

This provides a Linux OS running alongside Windows. Both share the same hard drive (and can access each other’s files), and the clipboard supports copy-and-paste between the two quite naturally. (A small thing, perhaps, but one that I find quite

What it is not

This provides a “server”-style installation of Linux. There is no full Linux desktop. The main entry point to Linux is a text-only `bash` shell for entering Linux commands. The ability to inter-script (i.e., to launch Windows programs from Linux or vice versa) seems quite limited.

What it can be

Combined with an X server running under Windows, you can launch and run GUI programs from Linux.

In this document, I’ll walk you through the process of setting up a programming environment consisting of:

*   The basic Linux on Windows system.
*   An X server (to display Linux-based GUI programs on the Windows-managed display screen).
*   Java and C++ compilers
*   The Eclipse IDE

Now, yes, you can install [those compilers](../../Public/installingACompiler/index.html) and [that IDE](../../Public/installingAnIDE/index.html) directly in Windows, but some things just seem to me to run more “smoothly” in Linux. And if your PC is a different version of Windows (or not Windows at all), those other guides are where you should go.

And there’s nothing wrong with trying out both approaches.

1 Get the Windows 10 Subsystem for Linux
========================================

You’ll find the instructions [here](https://msdn.microsoft.com/en-us/commandline/wsl/about), but in case that page moves or disappears, the summary is:

1.  Make sure you have Windows 10, 64-bit, and that you have been keeping it updated. (As of 6/1/2017, Microsoft says that you should have update 14393.0 or later).
    
2.  Turn on Developer Mode: Open `Settings` -> `Update and Security` -> `For Developers` and select the “Developer mode”. Close the `Settings` window.
    
3.  Enable Linux for Windows: From the taskbar, search for “Turn Windows features on or off”. Select “Windows Powershell 2.0” (if not already selected) and select “Windows Subsystem for Linux (beta)”. Click `OK`.
    
4.  Back at the taskbar, search for “ubuntu”. You should see that “Ubuntu App” has been installed. Select it to run `bash`.
    
    Try some simple Linux commands such as `ls`, `cd`, and `pwd`. You’ll find that your Windows lettered disc drives are available under ‘/mnt’. For example, your `C:` drive is `/mnt/c`.
    

1.1 sudo apt-get
----------------

Before we continue on, let’s introduce a couple of the programs that you will be using through the rest of this process.

*   `apt-get` is the program you use to update your Linux software and to install or uninstall software packages.
    
    `apt-get`, however, can only be run by a Linux administrator. Now, _you_ are the administrator for the Linux OS you have just installed, but for safety’s sake you _run_ most programs as an ordinary, non-administrator user.
    
*   `sudo` is a command that says “run the following as the administrator”.
    

For example, the following sequence is how you update your Linux software:

    sudo apt-get update
    sudo apt-get upgrade
    

The first command actually fetches the latest information about what updates are available. The second installs the updates. The “sudo” in front causes them to be run as the administrator. When you give the first “sudo” command, you will be prompted for your password to prove that you really are the account owner. After that, `sudo` remembers your identity for a short period of time, so you can give multiple `sudo` commands in a row, only asked for your password once, as long as you don’t take too much time in between.

Go ahead and give those two commands now.

> Try to give those two commands on a fairly regular basis so your Linux OS stays up to date. You’ll actually be notified when you start a new `bash` session if there are updates awaiting.

Close your `bash` session for now by giving the command

    exit
    

2 Installing X
==============

Now let’s work on displaying graphics form your Linux programs.

The first step is to get an X server package. Most X server packages put a heavy emphasis on connecting to remote machines, but we are going to use this one to serve programs running on the same local machine.

*   I find the [VcXsrv](https://sourceforge.net/projects/vcxsrv/) works very well in this setup.
    
    > Warning: X2Go, our favorite way to [remotely log in to the CS Dept Linux servers](https://www.cs.odu.edu/~zeil/cs252/latest/Public/xwinlaunch/index.html#x2go-client) also uses VcXsrv. This should not be a problem, but there is a bug in some versions of VcXsrv that can cause our intended use of VcXsrv to clash with X2Go.
    > 
    > If you are a frequent user of X2Go, I recommend using one of the other X packages listed below instead.
    
*   [XMing](http://www.straightrunning.com/XmingNotes/) seems to be a good choice as well.
    
    *   Get the public domain release for “Xming” and for the “Xming-fonts”.
*   [Cygwin/X](https://x.cygwin.com/) also works quite well, and if we were using the CygWin ports of the `g++` compiler, it would be my top choice. But since we are going to get those in the Windows Subsystem for Linux, it seems unnecessary to add the entire CygWin “POSIX inside Windows” layer.
    
    *   By default, CygWin/X accepts X connections from outside, but not from programs running on the same machine. The latter is exactly what we want to use it for. When launching CygWin/X, add “`-listen tcp`” to the “Additional parameters for X server” box.
        

1.  So, go to one of those X server websites, download the installer, and run it to install an X server.
    
    Then run the Xlaunch program to launch the server.
    
    > Get in the habit of running this X server just before you open a Windows `bash` session. It won’t do much of anything when you run it, other than add a new icon in the task bar tray. It is a _server_, a program that sits quietly and waits until some other _client_ program calls upon it.
    
2.  Run Windows `ubuntu`.
    
3.  We’re going to test your ability to run X GUI applications from within Linux and to see the results in Windows. So first we need to install an X application. In `bash`, give the command
    
        sudo apt-get install xterm   
        
    When the installation is complete, test it by giving the command
    
        DISPLAY=:0 xterm &  
        
    (For CygWin/X, you may need to change the “:0” to “:0.0”.)
    
    You should see a new window open up. This is another window in which you can issue Linux commands. It is, however, a Linux/X application, using X to render its general appearance, menus, etc. You can see this by holding the Ctrl key and then left- or right- clicking on the `xterm` window. Each mouse button will pop up a different menu.
    
4.  The `DISPLAY=` part of the earlier command is the way that we tell an X \_client" program (in that case, `xterm`) where to find an X server to handle drawing things and on which of many possible screens we want to draw (from among many that server might be managing). In this case, the `DISPLAY` value is pretty simple, because we are not connecting to a remote machine and we’re using using the default screen.
    
    Still, we don’t want to have to add that to every command. So let’s set that as the default for our Linux applications.
    
    Do
    
        nano ~/.bashrc
        
    Add the following line to the end:
    
        export DISPLAY=:0
        
    Save that and exit from `nano`.
    
    Exit from `bash`. And then restart it. Now
    
        xterm &
        
    should open an `xterm` window without your needing to supply the `DISPLAY` preamble.
    

3 Installing the Compilers
==========================

Now we’re ready to start installing some real software. You will need

*   The Java run-time system, because the IDE we will use, Eclipse, actually runs in Java. While we’re at it, we might as well get the whole Java compiler, in preparation for future programming projects in that language.
    
*   The `g++` compiler for C++.
    
*   The `gdb` debugger.
*   The `make` project build system.

We’ll install all of these with the following `apt-get` commands:

    sudo apt-get install default-jdk
    sudo apt-get install g++
    sudo apt-get install gdb
    sudo apt-get install make
    

Once those are done, verify your installation by typing the following in `bash`:

    java -version
    g++ --version
    gdb --version
    make --version 
    

Each should respond with an identifying message making clear that the software is installed and running.

4 Installing Eclipse
====================

Next up is the Eclipse IDE. We could install this with an `apt-get` command also, but the version in the Ubuntu respository seems to lag far behind the Eclipse project releases.

*   Do
    
            sudo apt-get install libgtk-3-0
        
    
    This adds a graphics library required for (the 2019 versions of) Eclipse.
    2.  Get Eclipse for your platform from the [Eclipse Foundation](https://www.eclipse.org/downloads/). Because you are browsing from Windows, it will try to push the Windows version of Eclipse at you by default. Use the “Download Packages” link (beneath the large “Download 64 bit” button). From there you can select the Linux 64 bit installer instead of the Windows installer.
    
    You’ll also have a choice of various pre-packaged forms of Eclipse.
    
    1.  If all that you are interested in is C++, scroll down to the “Eclipse IDE for C/C++ Developers” and download the installer for 64-bit Linux.
        
    2.  If you want to reserve the possibility of doing Java programming in the future, scroll down to the “Eclipse IDE for Java Developers” and download the installer for 64-bit Linux.
        
3.  What you will receive from this download is a `tar.gz`package. This is a compressed archive (similar to a `.zip` file).
    
    I will assume that you downloaded this into your Windows `Downloads` directory. In `bash`, copy this into your Linux home directory:
    
        cd ~
        cp /mnt/c/Users/your-Windows-login-name/Downloads/eclipse-java-*.tar.gz .
        tar xvzf  eclipse-java-*.tar.gz
        ls    
    
    You should see a new `~/eclipse/` directory. Your Eclipse installation is in there.
    
4.  Type
    
    ~/eclipse/eclipse &
    
    to launch Eclipse.
    
5.  **If You Installed the Eclipse for Java Developers**
    
    If you installed the Java Eclipse rather than the C/C++ Developer’s version, we need to add the C++ support.
    
    From the Eclipse Help menu, select `Install new software...`.
    
    Search “All Available Sites” for “C++” (Note, the search is _really_ slow. Be patient.) or scroll down to “Programming Languages”.
    
    Select:
    
    *   “C/C++ Development Tools”
    *   “C/C++ Library API Documentation…”.
    *   “C/C++ Unit Testing Support”
    
    and follow the on-screen instructions to install those.

You should now be able to create C++ projects in Eclipse.

If you need some help, revisit the Eclipse sections of [IDEs for Compiling under X](https://www.cs.odu.edu/~zeil/cs252/latest/Public/progdevx/index.html#eclipse) and [Debugging under X](https://www.cs.odu.edu/~zeil/cs252/latest/Public/debugx/index.html#the-eclipse-debugger) from CS252.

5 Optional packages
===================

The main purpose of this document was to set up your programming environment. But since you now have a working Linux installation running under Windows 10, here’s a few optional packages you might want to consider installing.

Each of the packages named here can be installed via the `apt-get` command

*   Editors: Sometimes you may want to fire up an editor other than Eclipse. Your Linux installation already comes with `vim` (a Unix editor with a long history, a popular following, but a very steep learning curve) and `nano`, a basic, non-GUI editor that’s great when you only need to change a few lines of text here and there (as [above](#usingNano)). If you want something else, consider:
    
    *   `scite`: a simple, intuitive GUI-based editor (described [here](http://www.scintilla.org/SciTE.html) that is lightweight (in disk requirement) but still offers syntax highlighting for many programming languages.
        
        There are others that are more popular (e.g., `gedit`, but many of those rely on a host of other libraries that are normally provided by the full Linux desktop, and since you aren’t running a Linux desktop, `apt-get` would need to download and install many tens of megabytes of those other libraries).
        
            sudo apt-get install scite

    *   `emacs`: Definitely not lightweight, but if you have gotten used to it…
        
            sudo apt-get install emacs
            
        
*   File Explorers: If you really dislike `cd`ing around just to examine your files and directories, you can have a file explorer without installing a full desktop, or even an appreciable fraction of the libraries needed to support a desktop.
    
    *   `mc`: short for [Midnight Commander](https://en.wikipedia.org/wiki/Midnight_Commander), this is _very_ lightweight and actually runs in text mode - it does not open in a separate X window.
            
            sudo apt-get install mc
            
    *   `xfe`: a more conventional, GUI-based (X windows) file explorer, but still very lightweight. It is described [here](http://roland65.free.fr/xfe/index.php?page=home).
        
            sudo apt-get install xfe
            
      
