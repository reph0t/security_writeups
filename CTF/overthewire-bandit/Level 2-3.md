
> [!important]
> Username: `bandit2`
> 
> Password: `PK8fYLZg2hnHSz83plBL1iEPKdD3QToB`

---
### Level Goal

The password for the next level is stored in a file called `--spaces in this filename--` located in the home directory

#### **Commands you may need to solve this level**

`ls , cd , cat , file , du , find`

---
## WALKTHROUGH
In this level, the file name contains spaces: `--spaces in this filename--`. When working with files that have spaces in their names, you need to handle them carefully.

One of the easiest ways to handle spaces in filenames is by wrapping the filename in double quotes (`""`). This tells the shell to treat the entire string (including spaces) as a single entity.

**Solution**

`cat./"--spaces in this filename--"`

Executing this command will output the contents of the file, which contains the password for the next level.

<img width="739" height="83" alt="image" src="https://github.com/user-attachments/assets/7783429f-a782-4de7-9a24-eb8a5d9725f8" />

