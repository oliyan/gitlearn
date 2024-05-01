## Connect to IBMI via VS Code

## Open the PASE Terminal.
## Set Shell to BASH
![alt text](image.png)

## Set Open Source path variables
- Create a new file called .profile in your home folder by issuing the command `touch .profile`
- Open the file `.profile` using VS Code's IFS Browser
- Copy paste the below content on the `.profile` file. 
```bash
PATH=/QOpenSys/pkgs/bin:$PATH
export PATH PASE_PATH
```