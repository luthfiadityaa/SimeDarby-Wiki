# A. Clone Repository

## A.1. Git

## A.2. SSH-KEY
  
Generate an RSA key pair from Command Prompt on Windows using built-in tools.

### 1.  Create
Option 1: Using ssh-keygen (Windows 10/11 with OpenSSH installed)
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

### 2. Adding
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

### 3. Testing

Test your SSH connection
Run:
```
ssh -T git@ssh.dev.azure.com
```

Expected response:

```
You've successfully authenticated, but Git does not provide shell access.
```