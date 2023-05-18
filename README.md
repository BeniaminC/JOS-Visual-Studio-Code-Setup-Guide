# jos_vscode_setup_guide (INCOMPLETE AS OF NOW)
How to setup Visual Studio Code to work with JOS.

Below is what it looks like
![Alt text](images/jos_vscode_debugging.png)

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
    sudo apt-get install -y git && sudo apt-get install -y qemu && sudo apt-get install -y qemu-system-i386 && sudo apt-get install -y gcc-multilib
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

# #Step 5: Setting Up Tasks

We now need to set up tasks for building our kernel, and also to launch our qemu.

In our `.vscode/tasks.json`, we need to first create a task to build our kernel using the `make` command:

```json
        {
            "type": "shell",
            "label": "Build kernel",
            "command": "make --directory=${workspaceFolder}",
            "args": [
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        },
```

Then we need to create a qemu launch tasks with no graphics.

```json
{
            "type": "shell",
            "label": "Launch Qemu (no graphic)",
            "command": "make qemu-nox-gdb",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "dependsOn": "Build kernel",
            "isBackground": true,
            "problemMatcher": [
                {
                  "pattern": [
                    {
                      "regexp": ".",
                      "file": 1,
                      "location": 2,
                      "message": 3
                    }
                  ],
                  "background": {
                    "activeOnStart": true,
                    "beginsPattern": ".",
                    "endsPattern": "Now run 'gdb'",
                  }
                }
            ]

        }
```
Notice that it requires a **problem matcher** with an **end pattern** so that the build can know when the qemu has started. We need to let the prelaunch tasks know when the qemu has finished before it can attach. Notice that the makefile runs a prompt "Now run 'gdb'. That is the pattern we are trying to match in the terminal

![Alt text](images/Screenshot%202023-05-17%20202758.png)

> You can add other launches from the makefile such as `make qemu-nox`, `print-qemu` etc.

## Step 6: Setting Up Lauches

After we have successfully set up our tasks, we can launch gdb for debugging. We first create a **prelaunch task** to first run (QEMU), then we run gdb.

```json
{
            "name": "Debug Kernel",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceRoot}/obj/kern/kernel",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Change back to the workspace folder",
                    "text": "cd ${workspaceRoot}",
                    "ignoreFailures": true
                },
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "Launch Qemu (no graphic)",
            "miDebuggerPath": "/usr/bin/gdb",
            "miDebuggerArgs": "",
            "targetArchitecture": "x86_64",
            "customLaunchSetupCommands": [
                {
                    "text": "target remote localhost:1234",
                    "description": "Connect to QEMU remote debugger"
                },
                {
                    "text": "symbol-file obj/kern/kernel",
                    "description": "Get kernel symbols"
                },
                // {
                //     "text": "set architecture i8086",
                //     "description": "Sets the current architecture"
                // }
            ],
            "avoidWindowsConsoleRedirection": true
        }
```
## Step 7: Debugging the Kernel

Now you can finally start debugging. First place a breakpoint in the code to start debugging:

![Alt text](images/Screenshot%202023-05-17%20203320.png)

Then start debugging by pressing **F5**.

You track variables while debugging:

![Alt text](images/Screenshot%202023-05-17%20203509.png)

You can watch specific variables:

![Alt text](images/Screenshot%202023-05-17%20203717.png)

And even track the callstack:

![Alt text](images/Screenshot%202023-05-17%20203848.png)

Run commands in the **Debug Console** using `-exec <command>`. For example, below the command `-exec info registers` is run:

![Alt text](images/Screenshot%202023-05-17%20204049.png)

> You should still run GDB commands for the labs!
- `Ctrl-c`\
    Halt the machine and break in to GDB at the current instruction. If QEMU has multiple virtual CPUs, this halts all of them.
- `c (or continue)`\
    Continue execution until the next breakpoint or Ctrl-c.
- `si (or stepi)`\
    Execute one machine instruction.
- `b function or b file:line (or breakpoint)`\
    Set a breakpoint at the given function or line.
- `b *addr (or breakpoint)`\
    Set a breakpoint at the EIP addr.
- `set print pretty`\
    Enable pretty-printing of arrays and structs.
- `info registers`\
    Print the general purpose registers, eip, eflags, and the segment selectors. For a much more thorough dump of the machine register state, see QEMU’s own info   registers command.
- `x/Nx addr`\
    Display a hex dump of N words starting at virtual address addr. If N is omitted, it defaults to 1. addr can be any expression.
- `x/Ni addr`\
    Display the N assembly instructions starting at addr. Using $eip as addr will display the instructions at the current instruction pointer.
- `symbol-file file`\
    (Lab 3+) Switch to symbol file file. When GDB attaches to QEMU, it has no notion of the process boundaries within the virtual machine, so we have to tell it which symbols to use. By default, we configure GDB to use the kernel symbol file, obj/kern/kernel. If the machine is running user code, say hello.c, you can switch to the hello symbol file using symbol-file obj/user/hello.
- `thread n`\
    GDB focuses on one thread (i.e., CPU) at a time. This command switches that focus to thread n, numbered from zero.
- `info threads`\\
    List all threads (i.e., CPUs), including their state (active or halted) and what function they’re in.

> QEMU represents each virtual CPU as a thread in GDB, so you can use all of GDB’s thread-related commands to view or manipulate QEMU’s virtual CPUs.
