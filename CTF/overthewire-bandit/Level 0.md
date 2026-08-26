## Part 1 - Logging In

### **Level Goal**
The goal of this level is for you to log into the game using SSH. The host to which you need to connect is bandit.labs.overthewire.org, on port 2220.

The username is bandit0 and the password is bandit0. Once logged in, go to the Level 1 page to find out how to beat Level 1.

**Commands you may need to solve**

- `ssh`

Before we can move on to Level 1, we need to access Level 0. The Level Goal instructs us to log into the game using a Linux tool called `ssh` (Secure Shell Protocol).

### **What is SSH?**
**SSH** is a network protocol used to connect to other hosts/servers 💻 --- 💻 in a unsecured network through a **secure connection**.

**SSH** connects and logs specified _destination_, which may be specified as either `user@hostname` or URI of the form `ssh://user@hostname:[port]`

The user must prove their identity to the remote machine using one of the several methods.

## **WALKTHROUGH**
On the Level Goal we are already give the credentials needed to connect to the server. Which are:

- Username: **bandit0**
- Password: **bandit0**
- Host: **bandit.labs.overthewire.org**
- Port: **2220**

Its all of a matter putting everything together as a command.

We are going to use method, `-p` to enter the specific port we want to connect to the server.

COMMAND: `ssh -p 2220 bandit0@bandit.labs.overthewire.org`

- `ssh`: This initiate the secure connection
- `-p 2220`: This specifies port 2220, which is the port used by the Bandit game server
- `bandit0@bandit.labs.overthewire.org`: This is the username(`bandit0`) and the host (`badnti.labs.overthewire.org`) to which we want to connect

When prompted, enter the password `bandit0` as shown below:

<img width="919" height="301" alt="image" src="https://github.com/user-attachments/assets/be00e9d0-114e-4976-9047-8eccf2bebd96" />

After entering the correct password, you will be logged into the Bandit server, where you'll see a welcome message and some rules regarding this level:

### **Login Confirmation**

To confirm that we have logged in successfully we can use commands `whoami` and `hostname` to verify our current username and hostname we are connected to:

<img width="862" height="274" alt="image" src="https://github.com/user-attachments/assets/d8f86bc0-acd1-4b20-9bc4-7b7c38b1d02b" />

This is a confirmation that we have succeeded and connecting the bandit host.
