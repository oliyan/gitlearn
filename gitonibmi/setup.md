## First things first
- Connect to IBMI via VS Code. You should knew that now already. If not, check [this](https://github.com/Programmersio-IBMi/vscode-integration/blob/main/README.md) link and come back here once you have connected your IBMi.  
- Open the PASE Terminal. Enter `Ctrl + Shift + J` once connected to the IBM I via VS Code and select *PASE* terminal
### Set Shell to BASH
  Either set it via VS Code. 
  ![alt text](image.png)
  <br>
  **OR**
  <br>
Enter the below command in the PASE terminal. *Don't forget to ==replace username==*
  ```bash
  /QOpenSys/pkgs/bin/chsh -s /QOpenSys/pkgs/bin/bash username
  ```

  - **Disconnect the IBMI and reconnect again for the BASH to take effect.**
  - Once connected, open up the PASE terminal again. If the shell is set to bash successfully, you should see the below screen
  ![alt text](image-1.png)

## Set Open Source path variables
In order to be able to run the linux commands without specifying the path name, we have to setup the Open Sourc Path Variables. Do the below steps to do so.
- Create a new file called .profile in your home folder by issuing the command `touch .profile`
- Open the file `.profile` using VS Code's IFS Browser
  ![alt text](image-2.png)
- Copy paste the below content on the `.profile` file. 
```bash
PATH=/QOpenSys/pkgs/bin:$PATH
export PATH PASE_PATH
```

## Install GIT via YUM
`yum install git`

## Connect a remote repository
**Setup the user name and email for your local git**
  ```bash
  git config --global user.name 'Ravisankar Pandian' #To add a user name for the git application.

  git config --global user.email ravisankar.pandian@programmers.io #To add email for the git application 
  ```
**Generate a public/private keypair.**
 Enter the below command in your PASE terminal. (make sure to enter the email id that you use in your github account)
   `ssh-keygen -t ed25519 -C "ravisankar.pandian@programmers.io"`
- Hit enter again to save the key pair at the default location itself. 
- Hit enter again (no passphrase is required)
- Notice the location of the public key and open it in your VS Code. 
 ![alt text](image-6.png)

**Copy the public key**
- Navigate to the same folder in your VS Code as below and open the public key
   ![alt text](image-5.png)
- Copy all the contents of the file. We need to put that into our GitHub account.
  
**Create New SSH Key in your remote repository**
- Open https://github.com/settings/keys and click 'New SSH Key'. 
- Enter some title, Select the key type as "authentication key", paste the previously copied public key, and finally select 'Add SSH Key"
  ![alt text](image-7.png)
- Once added, you should see the below screen
  ![alt text](image-8.png)

**Connect to the remote repository**
- Now it is time for us to connect to a remote repo. I have setup a GitHub repository called *'gitlearn'* for this experiment and I am going to clone that. 

- You can create your own repo in your github account and copy the command to clone via SSH as below.
  ![alt text](image-9.png)

- FYI: This is the command that I just copied
`git@github.com:ravisankar-PIO/gitlearn.git`

- Go to the PASE Terminal and enter
`git clone git@github.com:ravisankar-PIO/gitlearn.git`
![alt text](image-10.png)
