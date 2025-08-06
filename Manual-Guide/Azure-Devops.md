[[_TOC_]]
# A. Clone Repository

## 1. Git

## 2. SSH-KEY (Command Prompt)
  
Generate an RSA key pair from Command Prompt on Windows using built-in tools. if you using eclipse to setup SSH-Key, please refer to this [register-ssh-key](https://dev.azure.com/Daifuku-SW/MY_SunwayWinstar/_wiki/wikis/MY_SunwayWinster.wiki/294/Quick-Start-guide?anchor=register-ssh-key) Wiki.

### 2.1.  Create
Using ssh-keygen (Windows 10/11 with OpenSSH installed)
Modern Windows versions come with ssh-keygen via OpenSSH.

🔹 Steps:
1. Open Command Prompt (cmd).
2. Run:
```
ssh-keygen -t rsa -b 2048 -f C:\Users\%USERNAME%\.ssh\my_rsa_key
```

🔹 Explanation:
- -t rsa → key type RSA
- -b 2048 → 2048-bit key (you can use 4096 for stronger encryption)
- -f → path to save the key (public + private)

🔹 Output:
- my_rsa_key → private key
- my_rsa_key.pub → public key

### 2.2 Adding
Ensure SSH Agent is running and your key is added
In Git Bash or PowerShell:

```
eval $(ssh-agent)  
ssh-add ~/.ssh/your_private_key
```
  
For example:

```
ssh-add ~/.ssh/id_rsa
```
      
On Windows (Command Prompt or PowerShell):

```
ssh-add C:\Users\YourUsername\.ssh\id_rsa
```

To check if it's already added:

```
ssh-add -l
```

### 2.3 [Register to Azure DevOps](https://dev.azure.com/Daifuku-SW/MY_SunwayWinstar/_wiki/wikis/MY_SunwayWinster.wiki/294/Quick-Start-guide?anchor=register-ssh-key)

### 2.4 Testing

Test your SSH connection
Run:
```
ssh -T git@ssh.dev.azure.com
```

Expected response:

```
You've successfully authenticated, but Git does not provide shell access.
```

## 3. [Clone](https://dev.azure.com/Daifuku-SW/MY_SunwayWinstar/_wiki/wikis/MY_SunwayWinster.wiki/294/Quick-Start-guide?anchor=clone-repository)

#B. Build Project
- Import all projects on Eclipse to build.

### 1. Configure hosts file
- Open hosts file and copy the setting as listed below.  
C:\Windows\System32\drivers\etc\hosts
```
127.0.0.1       localhost rmiserver jasper-printserver
127.0.0.1       DBSERVER as21server
127.0.0.1       AGC01
127.0.0.1       MOSA001
```

### 2. [Create Database](https://dev.azure.com/Daifuku-SW/MY_SunwayWinstar/_wiki/wikis/MY_SunwayWinster.wiki/294/Quick-Start-guide?anchor=create-databse)

### 3. [Start Process](https://dev.azure.com/Daifuku-SW/MY_SunwayWinstar/_wiki/wikis/MY_SunwayWinster.wiki/294/Quick-Start-guide?anchor=start-processes)

#C. [Pull Request (PR) - Overview](https://dev.azure.com/Daifuku-SW/MY_SunwayWinstar/_wiki/wikis/MY_SunwayWinster.wiki/758/Pull-Request-(PR))