<h1 align=center> Modern DevOps Practices on IBMi</h1>

<p align = center> Practical Guide </p>


><h3> Table of Contents
</h3>

- [Pre-requisites](#pre-requisites)
  - [Connect to IBMI from VS Code](#connect-to-ibmi-from-vs-code)
  - [Set Shell to BASH](#set-shell-to-bash)
  - [Set Open Source path/env variables](#set-open-source-pathenv-variables)
    - [Setup for PASE/SSH terminal](#setup-for-pasessh-terminal)
    - [Setup for IBMi Green screen](#setup-for-ibmi-green-screen)
  - [Verify the setup](#verify-the-setup)
- [Install GIT](#install-git)
- [Setup GITHUB](#setup-github)
- [Setup Jenkins](#setup-jenkins)
    - [Download the Jenkins install file.](#download-the-jenkins-install-file)
    - [Start Jenkins](#start-jenkins)
    - [Initial Configuration](#initial-configuration)
    - [Create a New Job in Jenkins](#create-a-new-job-in-jenkins)
    - [Configure the Job to connect to GitHub](#configure-the-job-to-connect-to-github)
- [Install GitBucket on IBMi](#install-gitbucket-on-ibmi)
- [Footnotes/References](#footnotesreferences)
- [Further Research](#further-research)
  - [GitLab](#gitlab)
  - [PM2](#pm2)
  - [Service-Commander](#service-commander)
  - [Gmake or BOB?](#gmake-or-bob)
  - [Chroot](#chroot)
  - [Test Cases](#test-cases)
  - [Migrate to IFS](#migrate-to-ifs)
- [Conclusion](#conclusion)

---
<img src="initsetup.jpg"  width="50">

# Pre-requisites
1. A GitHub Account
2. VS Code Editor
3. An IBM i server running 7.2 or above
4. Some patience and desire to learn something new.

## Connect to IBMI from VS Code
>*because, it is easy to execute shell commands and edit IFS files in VS Code!*
- Connect to IBMI via VS Code. You should knew that now already. If not, check [this](https://github.com/Programmersio-IBMi/vscode-integration/blob/main/README.md) link and come back here once you have connected your IBMi.  
- Enter `Ctrl + Shift + J` (once connected to the IBM I via VS Code) and select **PASE** terminal.


## Set Shell to BASH
>*because, the default shell is very limiting and irritating!*

Either set it via VS Code. 
  
  ![alt text](image.png)
  

  **<p align="center">OR</p>**
  
Enter the below command in the PASE terminal. *Don't forget to ==replace cecuser with your username==*
  ```bash
  /QOpenSys/pkgs/bin/chsh -s /QOpenSys/pkgs/bin/bash cecuser
  ```

## Set Open Source path/env variables
>*because, we need to tell the IBMI where to locate the open source linux commands.*

In order to be able to run the linux commands without specifying the location of the command, we have to setup the Open Source Path/Environment Variables. Do the below steps to do so.
### Setup for PASE/SSH terminal
Follow the below steps if you decide to run the applications from the PASE/SSH Terminal
- Navigate to your home folder by entering `cd ~`
- Enter command `touch .profile` in order to create a new file called *.profile*
- Open the file `.profile` using VS Code's IFS Browser
  
  ![alt text](image-2.png)
- Copy paste the below content on the `.profile` file.

```bash
export PATH=/QOpenSys/pkgs/bin:$PATH
export JAVA_HOME=/QOpenSys/QIBM/ProdData/JavaVM/jdk17/64bit
export JENKINS_HOME=/home/CECUSER/jenkins
export GITBUCKET_HOME=/home/CECUSER/gitbucket
```
>**Explanation:**
>#1: The open source linux commands are available in the path `/QOpenSys/pkgs/bin`, so we are appending that location to the already available `$PATH` variable. 
>#2: The default JAVA version in IBMi sometimes would be 8. But Jenkins require version 11 or above. So we will tell IBMi to use the latest version of JAVA (17 in our case) for running Jenkins.
>#3: We are setting up the Jenkins' application on a folder called 'jenkins'. It provides better management of application, such as the whole application can be uprooted and planted in another location if required. 
>#4: Similarly, we are setting the Gitbucket's application on a folder called 'gitbucket'

### Setup for IBMi Green screen
If you decide to start the application from the green screen, then you have to run these commands.
  ```js
  ADDENVVAR ENVVAR('JAVA_HOME') VALUE('/QOpenSys/QIBM/ProdData/JavaVM/jdk17/64bit') LEVEL(*SYS) REPLACE(*YES)
  ADDENVVAR ENVVAR('JENKINS_HOME') VALUE('/home/CECUSER/jenkins') LEVEL(*SYS) REPLACE(*YES)
  ADDENVVAR ENVVAR('GITBUCKET_HOME') VALUE('/home/CECUSER/gitbucket') LEVEL(*SYS) REPLACE(*YES)
  /* note: You need to have ALLOBJ or SECOFR authority to run these commands  */
  ```
  ![alt text](image-50.png)

  >Note: Either setup of PASE or the setup of Green screen is enough. But it doesn't hurt to setup the path/env variables at both the places.

 ## Verify the setup
 Once the initial setup is complete,
- **Disconnect the IBMI and reconnect again**
- Once connected, open up the PASE terminal again by entering `Ctrl+Shift+J` If the shell is set to bash successfully, you should see the below screen
![alt text](image-1.png)
- Run the below command to check whether the path variable has been setup correctly.
  ```bash
  echo -e '\n' $PATH '\n' $JAVA_HOME '\n' $JENKINS_HOME '\n' $GITBUCKET_HOME '\n'
  ```
  ![alt text](image-38.png)

---

<img src="gitlogo.png"  width="75">


# Install GIT 
 
Enter the command below in your PASE terminal.
`yum install git`

![alt text](image-51.png)
>*GIT has been installed successfully*
---

<img src="githublogo.jpg"  width="100">

# Setup GITHUB
Let's connect our IBMi with the GitHub and try pushing (a.k.a. updating our sources) directly to the GitHub Repository.

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
 ![alt text](image-52.png)

**Copy the public key**
- Navigate to the same folder in your VS Code as below and open the public key
  
  ![alt text](image-53.png)
- Copy all the contents of the file. We need to put that into our GitHub account.
  
**Create New SSH Key in your GitHub account**
- Open https://github.com/settings/keys and click 'New SSH Key'. 
- Enter some title, Select the key type as "authentication key", paste the previously copied public key, and finally select 'Add SSH Key"
  ![alt text](image-54.png)
- Once added, you should see the below screen
  ![alt text](image-55.png)

**Create a GitHub repository**
- Let's create a new empty repository in our GitHub account.
- Click on Repository >> Click New
  ![alt text](image-56.png)
- Enter the repository name, some meaningful description, set it public, add a README.md file and click create repository.
  ![alt text](image-57.png)
- Nice, we have our own GitHub repository now.
  ![alt text](image-58.png)
- Click on the green `<> Code` button, click on `SSH` and copy the URL
   ![alt text](image-59.png)
<br>
- FYI: This is the command that I just copied
`git@github.com:ravisankar-PIO/gitonibmi.git`

**Clone the GitHub Repository to your IBMi**
- Go to the PASE Terminal in VS Code and enter
`git clone git@github.com:ravisankar-PIO/gitonibmi.git`
Enter `yes` if it asks for anything about Fingerprint and Keys. Now we have successfully cloned the GitHub Repository to our IBMi

  ![alt text](image-60.png)

**Create a simple sqlrpgle program**
- Let's create an SQLRPGLE program which inserts a record into some file for every time it is called.
- Go to the PASE Terminal in VS Code and enter
- `cd gitonibmi` =>  *to navigate to our repository folder*
- `git init` => *to initialize the git repository*
- `touch buildr.sqlrpgle` => *to create a new SQLRPGLE program*
- Once created, open the same file in your VS Code editor via the IFS Browser.
  ![alt text](image-62.png)
<br>
- Copy paste the below code into the `buildr.sqlrpgle` file and save it.
```js
**free
dcl-s count int(10);
dcl-s note varchar(50);

exec sql SELECT COUNT(*) INTO :count FROM ravi.buildpf;
count += 1;
note = 'Build# ' + %char(count);
exec sql INSERT INTO ravi.buildpf (note) VALUES (:note);

*inlr = *on;
```

**Commit the program**
- Enter the below commands one by one. Read below for explanation
```bash
git add buildr.sqlrpgle
git commit -m "added build sqlrpgle program"
git push
```
>*Explanation*
> * `git add` => we're telling the GIT that we're adding a new file in our repository.
> * `git commit` => we're telling the GIT that we want to commit our changes that we've made to the previously added files. The commit message describes the purpose of this change. 
> *  `git push` => we're telling the GIT to push the changes to the remote repository (GitHub) 
> * *Tip: between every command, you can check the status by issuing 'git status' command*

- Once the changes are pushed, you should see a message like something below
  ![alt text](image-63.png)

- Head over to the GitHub repository to check if the changes are updated there.
  ![alt text](image-64.png)

**Congratulations! You have successfully modernized the IBM i development to GitHub.**

---
<img src="jenkinslogo.jpg"  width="150">

# Setup Jenkins
Jenkins is a java based application so it can be run pretty much anywhere. We will setup Jenkins in our IBMi and serve it from the IBMi's IP address itself.

### Download the Jenkins install file.
Run the below commands in your PASE terminal to download the `jenkins.war` via `wget` command.  
```bash
cd ~ 
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
```
![alt text](image-65.png)
> *Jenkins is downloaded successfully!*
<br>

### Start Jenkins
Launching the Jenkins app is nothing but launching the `jenkins.war` file via a JAVA command with correct parameters. It can be started via multiple methods. For the very first run, we will follow method-1
*(Notice that I am using the **port# 9095**)*
* **Method-1:** Start directly in an interactive SSH sesion
  Head over to the green screen and issue the command below. 
  ```bash
  java -jar /home/CECUSER/jenkins.war --httpPort=9095
  ```
<br>

* **Method-2:** Start as a batch Job in Green Screen
  Head over to the green screen and issue the command below. 
  ```js
  SBMJOB CMD(QSH CMD('java -jar /home/CECUSER/jenkins.war --httpPort=9095')) JOB(JENKINS)
  ```
    <br>
* **Method-3:** Use Process Management tool like PM2 or Service Commander.
  Jump to [this section](#pm2) to view how to start the application via PM2.
  Jump to [this section]() to view how to start the application via Service-Commander.

 ### Initial Configuration
If all worked correctly, then a default admin password will be stored on the below location. Open the file `initialAdminPassword` and copy the contents of that file to your clipboard.
![alt text](image-67.png)


**Jenkins initial setup in browser**
Head over to the browser and type in the IP address of the IBMi followed by the port# that we defined earlier. In my case, it is `http://129.40.94.17:9095/`. Paste the admin password that we just copied a while ago to unlock Jenkins. 
![alt text](image-68.png)

Remember to select =="Install suggested plugins"==
*(**Note**: It will take some time to load the next screen. Don't click more than once, as it might end up in error)*
![alt text](image-24.png)
*plugins are currently loaded*
![plugins are loaded](image-15.png)

Let's create an Admin user which will be used to login to the Jenkins app from now on. 

```
UserName: ravi
Password: welcome
Email: ravisankar.pandian@programmers.io
```
![alt text](image-25.png)

*click on save and finish to complete the setup*
![alt text](image-26.png)

*Nice! we can start using the Jenkins now*
![alt text](image-17.png)

*If you see a notification at the top as given below, It is advised to have a separate node for building the code. But we will click dismiss for now and continue with our work*
![alt text](image-18.png)

<br>

### Create a New Job in Jenkins
- On the Dashboard, click on New Item
  ![alt text](image-69.png)
  <br>
- Enter the Job Name as `gitonibmi`, select `freestyle project` and click OK.
  ![alt text](image-71.png)

### Configure the Job to connect to GitHub
- In the Job Configuration Page, Click on the `Source Code Management` on the left, and select `GIT`. 
- Remember the Repository URL that we copied a while ago from our GitHub? We need to paste that over here. 
  ![alt text](image-72.png)

**Add a credential**
- Notice the `+Add` button under credential. Click on it to add a credential to connect to the GitHub securely.
  *Note: Sometimes, the button will load slowly. So don't press multiple times*
  ![alt text](image-73.png)
- Enter the details as below
  - Domain - Global (unrestricted)
  - Kind - SSH Username with Private key
  - Scope - Global
  - ID - anything you like
  - Private key - Select `enter directly` => click add => Open the private key from your ssh folder as below => copy the entire content => finally paste it on Jenkins window.
  ![alt text](image-74.png) <br> 
  - Then click add
  - Then, click on the credentials drop down, and select the one that has your user name.
   ![alt text](image-79.png)

**Add a Build Trigger**
- Click on `Build Triggers` on the left menu and check `Poll SCM`
  ![alt text](image-75.png)
- Enter the value as `* * * * *` (5 asteriks with spaces inbetween). This is a cron scheduler which will look for any changes made in our repository for every minute.
  ![alt text](image-76.png)

**Add Build Steps**
- Scroll down to find `Build Steps` and click on it, then select `Execute Shell`
  ![alt text](image-77.png)
- Enter the below PASE command 
  `system "CALL PGM(RAVI/BUILDR)"`
  This means whenever the GitHub repository is committed (i.e. updated), we will call a program called `buildr` 
  <br>
- That's it! We will save the Job Configuration now.
  ![alt text](image-78.png)

**Error!**
If you see the error as below and wondering what you did wrong? Then don't worry I am on your side too! Let's just proceed forward.
![alt text](image-80.png)  





a new file should be created for every build
![alt text](image-34.png)

![error-message](image-31.png)

![alt text](image-32.png)

files got created in the workspace directory
![alt text](image-33.png)

----


Start SSH agent eval $(ssh-agent -s)
Add SSh key to SSH agent ssh-add ~/.ssh/id_rsa.
Add github/gitlab to known host `ssh-keyscan github.com >> ~/.ssh/known_hosts`

---

# Install GitBucket on IBMi
Gitbucket is a JAVA based SCM tool which can be run on IBMi.


**Download the GitBucket installation JAR file first**
Run the below commands in your PASE terminal
```bash
cd ~ 
wget https://github.com/gitbucket/gitbucket/releases/download/4.40.0/gitbucket.war
```

**Start GitBucket using the Java Command**
Launching the GitBucket app is nothing but opening the `gitbucket.war` file via a JAVA command with correct parameters. It can be started via multiple methods. 

*(Notice that I am using the **port# 8085**)*
* **Method-1: (preferred)** Start as a batch Job in Green Screen
  Head over to the green screen and issue the command below. 
  ```js
  SBMJOB CMD(QSH CMD('java -jar /home/CECUSER/gitbucket.war --port=8085')) JOB(GITBUCKET)
  ```
    <br>
* **Method-2:** Start directly in an interactive SSH sesion
  Head over to the green screen and issue the command below. 
  ```bash
  java -jar /home/CECUSER/gitbucket.war --port=8085
  ```
<br>

* **Method-3:** Use Process Management tool like PM2
  Jump to [this section](#pm2) to view how to start the application.

---
# Footnotes/References

- Jenkins 
  - Refer [this](https://github.com/worksofliam/blog/issues/4), [this](https://pm2.keymetrics.io/docs/usage/quick-start/) and [this](https://www.youtube.com/watch?v=0O2Nz5duuzg) link to automate Jenkins using PM2.

  - [Getting started](https://devopscube.com/jenkins-2-tutorials-getting-started-guide/) with Jenkins.
  - Create Jenkins [pipeline](https://www.jenkins.io/doc/pipeline/tour/hello-world/)
  - [setup Jenkins](https://github.com/worksofliam/blog/issues/43) on IBMi

- GitBucket 
  - A VCS that can be hosted within IBMi. Click [here](https://github.com/gitbucket/gitbucket/wiki) to learn more.
- Source Orbit
  - Defintion of [Source Orbit](https://ibm.github.io/sourceorbit/#/) - a dependency management system
  - A [blog post](https://github.com/worksofliam/blog/issues/66) by Liam Barry that tells about the SO - SourceOrbit.
- IBMi - CI
  - A built in CI tool within IBMi. See it in action [here](https://www.youtube.com/watch?v=t-9nOyBjjCU)
  - Learn more about [IBMi-CI](https://github.com/IBM/ibmi-ci)

- Click [here](https://ibmi-oss-docs.readthedocs.io/en/latest/yum/3RD_PARTY_REPOS.html) to view about the third party Repos for IBMI

[Previous attempts](https://pixerenet1.sharepoint.com/:w:/r/sites/ProgrammersIO/_layouts/15/Doc.aspx?sourcedoc=%7BC6BB8D56-3508-48C7-BA9F-0976B312D379%7D&file=ci_cd_ibmi_Ashish%20(InProcess).docx&action=default&mobileredirect=true) - follow this for step by step with screenshot - by Ashish

[PIO's guide for OSSonIBMi](https://github.com/Programmersio-IBMi/OSSonIBMi)

[RPG-GIT-BOOK](https://github.com/worksofliam/rpg-git-book/blob/main/4-repository-host.md) - This is an excellent starting point for moving to GIT


---
# Further Research

<img src="https://logowik.com/content/uploads/images/gitlab8368.jpg"  width="150">

## GitLab
Since Gitlab is an open source alternative for GitHub, I wanted to check if it can be installed on the IBMi. But it didn't helped.

First, I **Installed the dependencies for GitLab**
`yum install -y curl policycoreutils-python openssh-server perl`

https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh
![alt text](image-27.png)

- Read more about Gitlab
  - [dependencies for GitLab](https://archlinux.org/packages/extra/x86_64/gitlab/)
  - [CICD Script to connect GitLab with IBMI](https://gitlab.com/JDubbTX/IBMiExamples/-/blob/main/.gitlab-ci.yml?ref_type=heads)
  - [Gitlab installation methods](https://docs.gitlab.com/ee/install/install_methods.html)
  - [Gitlab installation steps](https://docs.gitlab.com/ee/install/installation.html)
  
---
<img src="pm2logo.png"  width="350">

## PM2
PM2 is a process management app (built on Node.js) which is like an enhanced Task Manager for IBMi. It will be used to autostart, keep the node.js & java based apps persistent.

**Install PM2**
- Kill the Jenkins app first if already launched by entering `Ctrl+c` two times on the PASE terminal where the Jenkins is currently launched.
- Install NodeJS =>  `yum install nodejs14`

- Then install PM2 =>  `npm install pm2@latest -g`

- Add the location of the nodejs's binary to the path
  - Open the .profile file and add a new location to include in path
  ![alt text](image-29.png)
  - Save and close the `.profile` file. Disconnect from IBMi and connect again (for the changes to take effect).

**Configure PM2**
- Create a new file called `jen.json` in your home folder. This will be used to start the Jenkins app.
  `touch jen.json`
- Paste this content into that file as below.
```json
  {
  "apps":[
  {
      "name":"jenkins",
      "cwd":".",
      "script":"/QOpenSys/usr/bin/java",
      "args":[
          "-jar",
          "jenkins.war",
          "--httpPort=9095"
      ],
      "exec_interpreter":"",
      "exec_mode":"fork"
   },
   {
    "name":"gitbucket",
    "cwd":".",
    "script":"/QOpenSys/usr/bin/java",
    "args":[
        "-jar",
        "/home/CECUSER/gitbucket/gitbucket.war",
        "--port=8085"
    ],
    "exec_interpreter":"",
    "exec_mode":"fork"
 }
 ]
}
```
- Run this command to start the Jenkins => `pm2 start jen.json`
- Once started, we can view the started apps by => `pm2 ls`
  ![alt text](image-37.png)
- For some reason, I am **unable to end the application via PM2**

---
## Service-Commander

---

## Gmake or BOB?
*Following [this](https://ibm.github.io/ibmi-bob/#/getting-started/installation) article for BOB*

1. Install IBMi Repos =>   `yum install ibmi-repos`
2. Install BOB => `yum install bob`
faced an error

3. update yum packages => `yum update` and try again to install bob
4. Bob installed successfully
5. Let's test the build command by cloning a git repo.
   1. Create a library to save the build into
    `system "CRTLIB LIB(BOBTEST) TEXT('Better Object Builder test project')"`
    2. Clone the git repo (that already contains sample sources)
   `git clone https://github.com/ibm/bob-recursive-example`
   3. Change directory
  `cd bob-recursive-example`
   4. Set the environment variable to point the library that we just created
  `export lib1=BOBTEST`
   5. Run the build using,
   `makei build`
   ![alt text](image-35.png)

---

## Chroot
Chroot creates IFS containers for an IBM i Job i.e. PASE process
![alt text](image-40.png)
>Further information about chroot can be found [here](https://docs.google.com/presentation/d/1t78A1YZlr88aYuEM2638U0nQywW4VQh_4H2RCtayZQo/edit#slide=id.g736c0ce3c_0_0)
>A [blog post](https://www.krengeltech.com/2016/01/a-root-change-for-the-better/) about Chroot
>Another [one](https://www.krengeltech.com/2016/02/a-root-change-for-the-better-part-ii/)


---
## Test Cases
[unit test cases](https://github.com/worksofliam/IBMiUnit)

---
## Migrate to IFS
[A tool to migrate the source members to IFS path](https://github.com/worksofliam/IBMiUnit)

---
# Conclusion
- Pick the right tools required for the DevOps Practice.
  - Jenkins, IBMi-CI ***(new)***, GitLab CICID, GitHub Actions etc., for CI-CD
  - Gmake or BOB for building the code
  - Usage of Source Orbit ***(new)*** to resolve dependency conflicts
  - Right tool to setup unit test cases. 
  - Migrate the Sources from Members to IFS.
  - Self hosted or cloud based VCS. I vote for Self Hosted (GitBucket)
  
*May be a **self hosted GitBucket**, running along with **Jenkins**, which triggers **Source Orbit** for checking object dependencies, once resolved which uses **gmake** to compile the sources & uses **IBMi-CI** for setting up the pipeline would be an ideal setup.*


![alt text](image-42.png)

![alt text](image-43.png)

![alt text](image-44.png)

![alt text](image-45.png)

![alt text](image-46.png)

![alt text](image-47.png)

![alt text](image-48.png)

![alt text](image-49.png)