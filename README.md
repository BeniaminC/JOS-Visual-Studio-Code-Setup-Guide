# jos_vscode_setup_guide
How to setup Visual Studio Code to work with JOS.

I have been wanting to finish this guide, but I can't seem to get it to compile on my desktop. It compiles on my laptop just fine. Perhaps I need to install an older QEMU on my WSL.


# Setting Up Local Debugging with Windows Subsystem for Linux, Visual Studio Code, QEMU, and GDB
### By Beniamin Condrea

This is a step-by-step guide to setting up your local OS2 environment. This will be similar to the **Lab Setup** section on the website for your OS2 class. It will include those steps as well, in which some of those steps will be explicitly omitted.

This idea of this guide is too allow you to debug your kernel while still able to run commands as before. That way, you can have a nice interface for debugging your labs.

## Step 1: Windows Subsystem for Linux
WSL is Windows Subsystem for Linux. It allows your to run a linux system through windows
> Website is found here: https://learn.microsoft.com/en-us/windows/wsl/install
1. Install your favorite GNU/Linux distribution from the Microsoft Store (Ubuntu is the most common one)
2. In **PowerShell** or **Command Prompt**, run the follow command to install WSL:
   ```powershell
   wsl --install
   ```
   This command will enable the required optional components, download the latest linux kernel, set WSL 2 as your default, and install a Linux for you (Ubuntu as default, you can change this).
   > The command above will only work if WSL is not installed. Trying running
   > `
   > wsl --list --online
   > `
   > to see a list of available distros and run
   > `
   > wsl --install -d <DistroName>
   > `
   > to install a distro.

   By Default, the installed Linux distribution will be Ubuntu. You can change this with the `-d` flag:
   - Change Linux Distribution: `wsl --install -d <distibution Name>`
   - View list of Linux Distributions: `wsl --list --online`
   - Install additional distributions: `wsl --install -d <distibution Name>`

   > For Linux/Bash command line, you can instead run the executable: `wsl.exe --install -d <Distribution Name>` or list via `wsl.exe --list --online`

2. Restart your machine
3. Set up your linux username and password
    You can now access your WSL by selecting **Start** then opening **Ubuntu** or any other distribution you installed. Select **Ubuntu on Windows** then type in your username and password. This is specific for the Linux distribution and it has no effect on your Windows settings. This will allow you to run **sudo** commands (Super User Do)

4. Update and upgrade packages
    You should be in the habit of updating and upgrading packages in Linux by running the command below:
    ```Bash
    sudo apt update && sudo apt upgrade
    ```
5. Install necessary programs for this class:
    - QEMU
    - Git
    ```Bash
    sudo apt-get install -y git && sudo apt-get install -y qemu && sudo apt-get install -y qemu-system-i386
    ```
## Step 3: Setup Visual Studio Code for WSL
1. Install Visual Studio Code
2. Install VSCode extensions:
   - Remote - WSL
   - C/C++ (optional)
   - Git History (optional)
    > The **Remote Developer extension pack** allows you to run WSL, SSH, and a remote container for editing and debugging code with all the Visual Studio Code features.
3. Locate your home directory in Ubuntu, it should be located in `/home/<username>/`

![Alt text](images/ubuntu_home.png)

4. Once in the directory, run the code command `code .`. This command will open Visual Studio Code inside the current working directory.
    > Running the `code` command requires the remote development extension pack for WSL.

    This is where we will run our `git clone` command for our assignments in the *integrated terminal*.

```bash
git clone git@gitlab.enexploitable.systems:your-id/jos.git
cd jos
```

## Step 4: Setting Up Kernel Debugging

Now that we have properly set up our environment for our work, we need to create tasks for our JOS makefile. Because we are working locally, we can manually set our gdb port to a number, perhaper `1234`. In our **GNUmakefile**:


![Alt text](images/gdb%20port.png)


## Step 7: Debugging the Kernel