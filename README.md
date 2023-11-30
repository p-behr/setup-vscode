# setup-vscode
How to set up Visual Studio Code locally and connect to remote system

## Prerequisites
* Get your IBM i credentials from Yoda
* Follow the instructions to set up ACS

## Installs
Install the following on your Windows computer:
* Download & install Git for Windows - https://git-scm.com/download/win
* Download & install Visual Studio Code - https://code.visualstudio.com/download


## Set up Visual Studio Code

When you see %%IBM i Profile%% in the instructions, replace with _your_ **IBM i** user profile (replace the % too).  
When you see %%Windows Username%% in the instructions, replace with _your_ **Windows** user name (replace the %).  
Whenever you see shell commands that you need to enter, they will start with "$".  This is the command prompt that you will see in the terminal and is not part of the command; do not type the "$" as part of the commands!  

### Set the default shell
Open Visual Studio Code.  
Press F1 to open the command prompt (displays at the top of the screen).  
Type "Terminal: Select Default Shell" in the search bar and press Enter.  
Select "Git Bash" from the drop-down list.  

## Setting up SSH

### Generate local SSH key
Press Ctrl+\` to open the terminal inside Visual Studio Code (displays at the bottom of the screen).  
> The \` should be the key above the Tab key

Enter the following command to change your current working directory to your home:  
`$ cd ~`  
Print your current working directory with this command:  
`$ pwd`  
This should print something like `c/Users/YourWindowsUsername`  

Enter the following command to generate an SSH key:  
`$ ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa-ocean-ssh`  
Press Enter twice to leave the passphrase blank  
> `ssh-keygen` is the command to run (generate key)  
> `-t rsa` specifies the type of key to create  
> `-b 4096` specifies the number of bits in the key  
> `-f ~/.ssh/id_rsa-ocean-ssh` specifies the filename of the key  

### Copy your public SSH key to IBM i

#### SSH copy 
If password authentication is allowed you can copy your public key using SSH.  
Enter the following command to copy your public key to the OC Skunks IBM i:  
`$ ssh-copy-id -i ~/.ssh/id_rsa-ocean-ssh %%IBM i Profile%%@OCSKUNKS.oceanusergroup.org`  
When prompted, enter your IBM i password.  
> `ssh-copy-id` is the command to run (copy id)  
> `-i ~/.ssh/id_rsa-ocean-ssh` specifies the identity file  
> `profile@OCSKUNKS.oceanusergroup.org` is the user and remote server

#### ACS
You can use ACS to copy your public key to IBM i.  
Open System Configurations, highlight your system, and press Edit.  
Select the SSH Key setup tab, and press the "Copy SSH key(s) to server" button.  

#### SQL
```
CALL QSYS2.IFS_WRITE(
    PATH_NAME =>'/home/{{user_profile}}/.ssh/authorized_keys', 
    LINE => 'ssh-rsa [...]== user@device', 
    OVERWRITE => 'APPEND', 
    END_OF_LINE => 'NONE'
);
```

#### Other
Use RDi, nano, or some other editor to add the public key to the authorized_keys file on the server. 
The authorized_keys file is located here: /home/YOUR_PROFILE/.ssh  

### Test the SSH connection
Enter the following command to create an SSH connection to the OC Skunks IBM i:  
`$ ssh %%IBM i User%%@OCSKUNKS.oceanusergroup.org -i ~/.ssh/id_rsa-ocean-ssh`  
You should see a "bash-4.4$" prompt.  
`$ exit` will disconnect the connection.  


## Editing remote source files

### Download/Configure the SSH FS extension
Press Ctrl+Shift+X to open the Extensions pane (or click the icon in left-hand menu).  
Type "SSH FS" in the search box.  
Press Install for SSH FS extension.  

Press F1 to open the command prompt.  
Type "SSH FS: create a SSH FS configuration" and select the link from the drop-down list.  
Enter the following configuration:
* Name = "ocean_skunks_ssh_fs"  
* Location = "Global settings.json"  
Press Save button  

On the next screen enter the following configuration:  
* Label = "OCEAN Skunks /home"  
* Group = "OCEAN"  
* Host = "OCSKUNKS.oceanusergroup.org"  
* Root = "/home/%%IBM i Profile%%"  
* Username = "%%IBM i Profile%%"  
* Private key = "c:\Users\\%%Windows Username%%\\.ssh\id_rsa-ocean-ssh"  
Press Save button

### Open/Edit remote files
Press Ctrl+Shift+E to open the Explorer pane.  
Open the SSH FILE SYSTEMS group at the bottom.  
Right-Click on the "OCEAN Skunks /home" connection.  
Select "Connect as Workspace folder".  


## Troubleshooting
If you are having issues (i.e. SSH is still asking for a password) check the file permissions:  
chmod 755 ~  
chmod 700 ~/.ssh  
chmod 600 ~/.ssh/id_rsa  
chmod 600 ~/.ssh/id_rsa.pub  
chmod 600 ~/.ssh/authorized_keys  
