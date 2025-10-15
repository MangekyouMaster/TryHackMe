## Shell Fundamentals & Basic Scripting

This document covers fundamental interactions with the **Linux shell**, essential basic commands, an exploration of common shell types, and an introduction to writing simple shell scripts, aligning with the "Shell Fundamentals" learning objectives.

-----

## I. Interacting with the Linux Shell

Interacting with the shell involves issuing **commands** to the interpreter (like **Bash**) to perform tasks.

| Command | Full Name/Purpose | Description |
| :--- | :--- | :--- |
| `pwd` | **Print Working Directory** | Shows the absolute path of your current directory. |
| `cd` | **Change Directory** | Used to navigate between directories. |
| `ls` | **List** | Displays the contents (files and directories) of the current directory. |
| `cat` | **Concatenate** | Reads and displays the entire contents of a file to the standard output. |
| `grep` | **Global Regular Expression Print** | A powerful command used to **search** for a specific word or pattern within one or more files. <br> *Example:* `grep hello file.txt` |

-----

## II. Types of Linux Shells

A **shell** is a program that interprets your commands. Different shells offer different features and levels of user-friendliness.

### Checking Available Shells

| Command | Purpose |
| :--- | :--- |
| `echo $SHELL` | Displays the name and path of your **current shell**. |
| `cat /etc/shells` | Lists all installed and **available shells** on the Linux system. |
| `chsh -s /usr/bin/zsh` | Command to **permanently change** your default login shell. |

### Comparison of Popular Shells

| Feature | **Bash** (Bourne Again Shell) | **Fish** (Friendly Interactive Shell) | **Zsh** (Z Shell) |
| :--- | :--- | :--- | :--- |
| **Default?** | Default for most Linux distributions. | Not default. | Not default. |
| **User-Friendly** | Less user-friendly; traditional. | **Most user-friendly**; simple syntax. | Highly user-friendly with proper customization. |
| **Tab Completion** | Basic feature. | **Advanced**; gives suggestions based on history. | **Extensible**; highly advanced via plugins. |
| **Scripting** | Widely compatible; extensive documentation. | Limited scripting features. | Excellent level of scripting; combines Bash capabilities. |
| **Customization** | Basic level. | Good customization via interactive tools. | **Advanced** via the `oh-my-zsh` framework. |
| **Syntax Highlighting** | Not available by default. | **Built-in** feature. | Available via plugins. |
| **Unique Feature** | Maintains command `history` (use $\uparrow$/$\downarrow$ or `history` command). | **Auto spell correction** for commands. | **Auto spell correction** for commands. |

-----

## III. Shell Scripting Fundamentals

Shell scripting allows you to automate a series of commands. For **Bash** (which uses the $\text{.sh}$ extension), a script must have execution permissions and be executed with `./` to specify its location in the current directory.

### 1\. Script Creation and Execution

| Step | Command/Action | Purpose |
| :--- | :--- | :--- |
| **Create** | `nano first_script.sh` | Use a text editor (like `nano`) to create a file with a $\text{.sh}$ extension. |
| **Permissions** | `chmod +x first_script.sh` | Grants **execution permission** to the script file. |
| **Execute** | `./first_script.sh` | Runs the script. The `./` tells the shell to execute the file in the **current directory**. |

### 2\. Essential Script Components

#### A. The Shebang (`#!`)

Every script must start with a **shebang**, which specifies the **interpreter** to be used for executing the script.

```bash
#!/bin/bash
# Specifies that the script should be run using the Bash interpreter.
```

#### B. Variables and Input (`read`)

A **variable** is a storage container for values (like a name, URL, or file path). The `read` command captures user input.

```bash
#!/bin/bash
echo "Hey, what’s your name?"
read name         # Stores user input in the 'name' variable
echo "Welcome, $name"  # Access the variable's value using $
```

#### C. Loops (`for`)

**Loops** repeat a block of code a specified number of times or while a condition is true. The `for` loop is a common iteration mechanism.

```bash
#!/bin/bash
# The loop_script.sh example
for i in {1..10}; do
    echo $i # Prints the current value of 'i' in the range 1 through 10
done
```

#### D. Conditional Statements (`if/else`)

**Conditional statements** execute code blocks only if a specified condition is met.

```bash
#!/bin/bash
# The conditional_script.sh example
echo "Please enter your name first:"
read name
if [ "$name" = "Stewart" ]; then
        echo "Welcome Stewart! Here is the secret: THM_Script"
else
        echo "Sorry! You are not authorized to access the secret."
fi
```

#### E. Comments (`#`)

Comments are lines that are **ignored by the interpreter** but are vital for human understanding. Use a hash sign (`#`) followed by a space.

```bash
# This is a comment. It helps explain the purpose of the code below.
echo "This command will be executed."
```

-----

## IV. Practical Scripting Example: Locker Authentication

This script demonstrates combining variables, loops, conditional statements, and logical operators (`&&`) for a simple authentication process.

### Scenario:

A script to verify a user based on three inputs:

  * **Username:** `John`
  * **Company name:** `Tryhackme`
  * **PIN:** `7385`

### Script (`auth_script.sh`)

```bash
#!/bin/bash

# --- Variable Initialization ---
# Initialize variables to hold user input
username=""
companyname=""
pin=""

# --- Input Loop (Iteration) ---
# Loop to ask for 3 distinct pieces of information
for i in {1..3}; do
    # Use conditional statements (elif) within the loop to guide prompts
    if [ "$i" -eq 1 ]; then
        echo "Enter your Username:"
        read username
    elif [ "$i" -eq 2 ]; then
        echo "Enter your Company name:"
        read companyname
    else
        echo "Enter your PIN:"
        read pin
    fi
done

# --- Authentication Check (Conditional Logic) ---
# Check if ALL entered details match the required values using the AND operator (&&)
if [ "$username" = "John" ] && [ "$companyname" = "Tryhackme" ] && [ "$pin" = "7385" ]; then
    echo "Authentication Successful. You can now access your locker, John."
else
    echo "Authentication Denied!!"
fi
```
