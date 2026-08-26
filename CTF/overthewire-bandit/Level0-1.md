### **Level Goal**

The password for the next level is stored in a file called readme located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH (on port 2220) to log into that level and continue the game.

**Commands you may need to solve this level**

`ls , cd , cat , file , du , find`

---
## **Understanding the File System**

Before we proceed, it’s helpful to understand the file system we’re navigating. If you've used your system’s terminal before, you’ll recognize that the server has a file hierarchy system (which can vary slightly based on the operating system).

When you run a command like ls (which lists files and directories), you’ll see important files and directories that are part of the system's structure. These files allow the system to run various applications and boot correctly.

For beginner users, it’s important not to modify or delete these native files unless you fully understand what you’re doing, as they can be crucial for system operation. Every technology relies on a file system as a reference to execute commands and manage operations.

---
## **WALKTHROUGH**

Now that we’ve briefly covered the file system, let's focus on the task at hand. According to the Level Goal, there is a file called readme in the home directory that contains the password for the next level.

You’re given several commands to help find and retrieve this password:

- `ls` - Lists all the contents in the current directory.
- `cd` - Changes directories, allowing you to navigate through the file system.
- `cat` - Concatenates and prints the content of a file to the terminal.
- `file` - Determines the type of a file (e.g., text, binary, etc.).
- `du` - Displays disk usage information for files and directories.
- `find` - Searches for files in a directory hierarchy.

With these commands available, let’s choose the best ones to help us retrieve the password.

### **Step 1: List the Directory Contents**

Use the `ls` command to see what’s inside the current directory (your home directory).

This will list all the files and directories in your current location.

<img width="932" height="833" alt="image" src="https://github.com/user-attachments/assets/19c3c369-392d-4df0-8c9d-2f1435bb1285" />

As you can see, there is indeed a file named `readme`.

### **Step 2: Display the File Contents**

Now that we've confirmed the `readme` file is present, we can use the `cat` command to display its contents and reveal the password.

`cat readme`

This will output the content of the file, including the password for the next level.

<img width="1076" height="252" alt="image" src="https://github.com/user-attachments/assets/9aceb42f-3e1b-4a5a-a6eb-b0518772ea17" />


Congratulations! You now have the password for **Bandit1**.

> [!warning] 
> The passwords you find for each level will not be saved automatically. It’s strongly recommended that you save them in a document for future reference. If you lose track of a password, you’ll have to start over from Bandit0, so make sure to document your progress!

