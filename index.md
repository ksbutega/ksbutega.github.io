Setting Up Windows WSL2 with XFCE Desktop
========================================================
*Updated April 13th 2021*
## Install WSL
### 1. Install Windows Subsystem for Linux (WSL)
--------------------------------------------

First we need WSL 1 and we can do this by either running

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
``` 

in a PowerShell or Windows Terminal as an administrator, or by going to ```Control Panel/Programs/Programs and Features``` and then in the left sidebar click 'Turn Windows features on or off'. On the list that comes up, select the "Microsoft Windows Subsystem for Linux"

### 2. Check Requirements for WSL2
To update to WSL 2, you must be running Windows 10.

    * For x64 systems: Version 1903 or higher, with Build 18362 or higher.
    * For ARM64 systems: Version 2004 or higher, with Build 19041 or higher.
    * Builds lower than 18362 do not support WSL 2. Use the Windows Update Assistant to update your version of Windows.

To check your version and build number, select Windows logo key + R, type winver, select OK. (Or enter the ver command in Windows Command Prompt). Update to the latest Windows version in the Settings menu.


### 3. Enable "Virtual Machine Platform"

In order to run WSL2, you need to enable the Virtual machine platform in the Windows Features. You can do this again by either running the following command in a PowerShell or Windows Terminal as admin
```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

or by going to ```Control Panel/Programs/Programs and Features``` and then in the left sidebar click 'Turn Windows features on or off'. On the list that comes up, select the "Virtual Machine Platform" and click OK.

### 4. Download and Install the WSL2 Kernel Update

Download and install [WSL2 Kernel Update](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi). Check [here](https://docs.microsoft.com/en-us/windows/wsl/install-win10) to see if there is a new link or new instructions

### 5. Set WSL2 as your default version
You can do this by running
```powershell
wsl --set-default-version 2
```
in Windows Terminal or PowerShell.

## Install Linux Distribution and xfce4 Desktop
### 1. Install Linux Distribution
Go to the Microsoft Store and install the Linux distribution you want (i.e. Debian). You can install multiple distributions. Note that the distribution is not actually installed until you Launch it for the first time. So, launch the distribution you want, set up the username and password and you are set to go.

### 2. Set your distribution to WSL1 or WSL2

You can check the WSL version assigned to each of the Linux distributions you have installed by opening the PowerShell command line and entering the command (only available in Windows Build 18362 or higher): ```wsl -l -v```
```powershell
wsl --list --verbose
```
To set a distribution to be backed by either version of WSL please run:
```powershell
wsl --set-version <distribution name> <versionNumber>
```
Make sure to replace <distribution name> with the actual name of your distribution and <versionNumber> with the number '1' or '2'. You can change back to WSL 1 at anytime by running the same command as above but replacing the '2' with a '1'.

### 3. Update the distribution
Now, launch the distro you chose, and run a good old
```shell
sudo apt-get update
sudo apt-get upgrade
```

### 4. Install XFCE4
This depends on the distribution you chose, but if you chose Debian, this comes without a desktop environment, so you need to install it. You can choose anyone you like, but I like xfce4 because it is simple, functional and fast. To install it, you can just run
```shell
sudo apt-get install xfce
```
This installed the environment, however, now we need to tell it where to show it. In order to do that, we need to install an X server that displays whatever the distro is showing.

## Install and Run VcXsrv 
There are other version of X server, but VcXsrv seems to be the standard and I saw no reason to change it.

### 1. Download and Install VcXsrv
Download it [here](https://sourceforge.net/projects/vcxsrv/) and install it. If the link is broken... Google it. Or better DuckDuckGo it, although it doesn't sound as good.

### 2. Change Compatibility Options, Firewall Settings 
You can manually run VcXsrv every time by running XLaunch, which should be installed on your desktop by now. One thing that you will want to do is change some display option for the X server. To do this:
    1. Go to "C:\Program Files\VcXsrv" 
    2. Right-click on xlaunch.exe and go to Properties
    3. Go to the Compatibility tab and at the bottom to "Change High DPI settings"
    4. Select the "Override high DPI scaling behavior..." and make sure that the drop-down menu has "Application selected"
    5. Click OK and exit

Next go to Windows Defender Firewall and in the side menu, go to "Allow an app or feature through Windows Defender Firewall", and scroll down and enable "VcXsrv windows xserver" by selecting the Private and Public boxes. Click Ok and exit.

### 3. Run VcXsrv
Now, run XLaunch. You can automate the selection process but for now, I chose
* One large window
* Start no client
* Clipboard, Primary Selection, Native opengl, Disable access control

***NOTE THAT THE DISABLE ACCESS CONTROL IS A MUST!!!***

If you start getting the error later on. It's because you didn't disable access controls

```shell
/usr/bin/startxfce4: X server already running on display X.X.X.1:0.0
Authorization required, but no authorization protocol specified
xrdb: Resource temporarily unavailable
xrdb: Can't open display 'X.X.X.1:0.0'
Authorization required, but no authorization protocol specified
Authorization required, but no authorization protocol specified
xfce4-session: Cannot open display: .
```


## Setup Linux Distro Display Options

### 1. Test VcXsrv Display Setup and WSL IP Address
Now, the servers gives our Linux distro a way to display its content. However, they are not aware of each other. In order to run a GUI program (xfce4 in this case) and tell it where to display, we need to run it using the following syntax
***NOTE: VcXsrv HAS TO BE RUNNING FOR THIS TO WORK. GO BACK AND USE XLAUNCH IF IT'S NOT***

```shell
DISPLAY=$(grep nameserver /etc/resolv.conf | awk '{print $2}'):0.0 startxfce4
```

Note that the part in parenthesis is what gives us the WSL IP for Windows. So the Linux distro knows to which IP and server to send the graphical information. This is the same that is obtained by running in Windows 
```powershell
ipconfig
``` 
and look for WSL ethernet adaptor.

If you set up everything correctly, you should see the desktop appearing on you WSL. If you get problems, first make sure that the firewall exception is setup, and second, if you see the following
```shell
/usr/bin/startxfce4: X server already running on display X.X.X.1:0.0
Authorization required, but no authorization protocol specified
xrdb: Resource temporarily unavailable
xrdb: Can't open display 'X.X.X.1:0.0'
Authorization required, but no authorization protocol specified
Authorization required, but no authorization protocol specified
xfce4-session: Cannot open display: .
```

you have not run VcXsrv with disable access control.

### 2. Setup Linux Distro for Future GUI Sessions
Now that you know that everything works, you don't want to add every time the DISPLAY=... part. So we can set up something that does it by default by going to the home folder and modifying the .bashrc file.
```shell
$ cd ~
$ nano .bashrc
```

Then append at the end the following lines
```
export HOST_IP=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')
export DISPLAY=$HOST_IP:0
```
and save. Now, you can avoid telling the distro which display to send the information to, and just use the commands as if you had a screen running. But still remember that you ***HAVE TO HAVE VcXsrv RUNNING***


Setting up a Programming Environment in WSL2
========================================================

[1 Installing the Compilers](#1-installing-the-compilers)

[2 Installing Eclipse](#2-installing-eclipse)

[3 Optional packages](#3-optional-packages)



## 1 Installing the Compilers

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

## 2 Installing Eclipse


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

## 3 Optional packages


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
            
      
