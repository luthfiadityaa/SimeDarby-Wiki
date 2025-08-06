#A. Clone Repository

##A.1. Git

##A.2. SSH-KEY
  
Generate an RSA key pair from Command Prompt on Windows using built-in tools.

###1.  Create
Option 1: Using ssh-keygen (Windows 10/11 with OpenSSH installed)
Modern Windows versions come with ssh-keygen via OpenSSH.
🔹 Steps:
    1.  Open Command Prompt (cmd).
    2.  Run:

<pre><code>ssh-keygen -t rsa -b 2048 -f C:\Users\%USERNAME%\.ssh\my_rsa_key</code></pre>

- Explanation:
-    -t rsa → key type RSA
-    -b 2048 → 2048-bit key (you can use 4096 for stronger encryption)
    *   -f → path to save the key (public + private)
🔹 Output:
    *   my_rsa_key → private key
    *   my_rsa_key.pub → public key

###2. Adding
Ensure SSH Agent is running and your key is added
In Git Bash or PowerShell:

<pre><code>eval $(ssh-agent)  
ssh-add ~/.ssh/your_private_key</pre></code>
  
For example:

<pre><code>ssh-add ~/.ssh/id_rsa</pre></code>
      
On Windows (Command Prompt or PowerShell):

<pre><code>ssh-add C:\Users\YourUsername\.ssh\id_rsa</pre></code>

To check if it's already added:

<pre><code>ssh-add -l</pre></code>

###3. Testing

Test your SSH connection
Run:
<pre><code>ssh -T git@ssh.dev.azure.com</pre></code>

Expected response:

<pre><code>You've successfully authenticated, but Git does not provide shell access. </pre></code>