# 🧪 Hands-on Lab: Master 50 Essential Linux Commands (Baby Steps)

## 🧩 Part 1 – File and Directory Management (Step by Step)

Let's start by creating a basic file and directory structure to practice all the following commands.

### 🔧 Step 1: Create a working folder
```bash
mkdir -p ~/lab-linux/arquivos
cd ~/lab-linux
```
- `mkdir -p` creates the main folder and subfolders if needed.
- `cd` enters the working directory.

### 📂 Step 2: Navigate and list content
```bash
ls
ls -l
ls -a
ls -lh
```
- `ls`: lists files.
- `ls -l`: shows details like permissions and size.
- `ls -a`: displays hidden files.
- `ls -lh`: shows readable sizes (KB, MB).

### 🔄 Step 3: Create empty files and navigate between folders
```bash
touch arquivos/um.txt
mkdir -p arquivos/subpasta
cd arquivos
pwd
```
- `touch` creates empty file.
- `mkdir` creates directory.
- `cd` enters folder.
- `pwd` shows current path.

### ⬆️ Step 4: Go back one level and create more folders
```bash
cd ..
mkdir backups
```
- `cd ..` goes back one folder.
- `mkdir backups` creates another folder.

### 🧹 Step 5: Remove empty directories
```bash
mkdir temp
rmdir temp
```
- `rmdir` removes empty directory.

### ❌ Step 6: Remove files and folders (be careful!)
```bash
rm arquivos/um.txt
rm -r arquivos/subpasta
rm -rf backups
```
- `rm` removes files.
- `rm -r` removes recursively.
- `rm -rf` forces removal without confirmation (careful!).

### 📋 Step 7: Create two files and copy them
```bash
mkdir arquivos
cd arquivos
touch a.txt b.txt
cp a.txt copia_de_a.txt
cp -r ../arquivos ../arquivos_backup
```
- `cp` copies files.
- `cp -r` copies directories recursively.

### 🚚 Step 8: Move and rename files
```bash
mv b.txt renomeado.txt
mv renomeado.txt ../
```
- `mv` moves or renames files.

### 🧼 Step 9: Check file type
```bash
file a.txt
```
- `file` shows the file content type.

---

## ✅ Part 1 Conclusion

Now you have a functional base to continue the next tests of file content and manipulation safely. All commands were executed on files you created yourself.

In the next part, we'll explore commands like `cat`, `less`, `head`, `tail`, `grep`, `sed`, `awk` etc., using the existing files and others we'll create as needed.

See you in the next part! 😉

---

## 📄 Part 2 – File Content Manipulation (Step by Step)

### 🪄 Step 1: Prepare files with content
```bash
cd ~/lab-linux
cd arquivos
echo -e "linha 1\nlinha 2\nlinha 3" > a.txt
echo -e "erro: falha\ninfo: ok\naviso: cuidado" > log.txt
echo -e "nome idade\njoao 30\nmaria 25" > dados.txt
```
- `echo -e` prints multiple lines to terminal, `>` saves to file.

### 📖 Step 2: View content with `cat` and `less`
```bash
cat a.txt
less log.txt  # Use q to exit
```
- `cat` shows content all at once.
- `less` allows navigation (arrows, scroll bar).

### 🔎 Step 3: View first and last lines
```bash
head -n 2 log.txt
tail -n 2 log.txt
tail -f log.txt  # Use Ctrl+C to exit
```
- `head` shows the beginning.
- `tail` shows the end.
- `tail -f` follows real-time updates.

### 🧠 Step 4: Search text with `grep`
```bash
grep "erro" log.txt
grep -i "AVISO" log.txt
grep -r "joao" .
```
- `grep` searches for text.
- `-i` ignores uppercase/lowercase.
- `-r` searches in subdirectories.

### ✏️ Step 5: Replace words with `sed`
```bash
sed 's/joao/JOÃO/g' dados.txt
```
- `sed` makes text substitutions.
- `s/old/new/g` replaces all occurrences.

### 📊 Step 6: Manipulate columns with `awk`
```bash
awk '{print $1}' dados.txt
awk '{print $2}' dados.txt
```
- `awk` allows working with text columns.

### 🔢 Step 7: Count lines, words and bytes with `wc`
```bash
wc -l dados.txt
wc -w dados.txt
wc -c dados.txt
```
- `wc` shows statistics: lines (`-l`), words (`-w`), bytes (`-c`).

### 🔀 Step 8: Sort lines with `sort`
```bash
sort dados.txt
```
- `sort` organizes lines in alphabetical or numerical order.

### 📑 Step 9: Compare files with `diff`
```bash
echo -e "nome idade\njoao 30\nmaria 22" > dados_v2.txt
diff dados.txt dados_v2.txt
```
- `diff` compares differences between files line by line.

---

## ✅ Part 2 Conclusion

In this step you learned how to read and manipulate text file content with the main terminal commands. We created simple files with `echo`, and all tests were performed based on files created in the previous step.

Ready to continue? In the next part we'll see system commands like `top`, `ps`, `kill`, `df`, `uptime`, and much more.

See you there! 🚀

[...]

---

## ⚙️ Part 3 – System Information and Management (Step by Step)

### 🧠 Step 1: View system information
```bash
uname -a
```
- `uname -a` shows kernel name, version, architecture and host name.

### 📊 Step 2: View processes in real time
```bash
top
```
- `top` displays active processes and CPU/memory usage in real time.

### 📈 Step 3: View processes with more details
```bash
ps aux
ps aux | grep bash
```
- `ps aux` shows all system processes with detailed information.
- `grep` filters specific processes (e.g. `bash`).

### 🔪 Step 4: Terminate processes manually
```bash
kill 1234
kill -9 1234
```
- `kill` sends signal to terminate a process.
- `-9` forces immediate termination.

> Tip: use `ps aux | grep program_name` to find the PID.

### 💽 Step 5: View disk and directory usage
```bash
df -h
du -sh ~/lab-linux
```
- `df -h` shows disk usage in all partitions.
- `du -sh` shows the size of specified folder.

### 🧠 Step 6: View memory usage and uptime
```bash
free -m
uptime
```
- `free -m` shows RAM and swap memory usage in MB.
- `uptime` shows how long the system has been running.

### 👥 Step 7: See who is logged into the system
```bash
who
w
```
- `who` lists logged in users.
- `w` shows logged in users and what they're doing.

### 🕓 Step 8: View command history and restart
```bash
history
sudo shutdown -h now
sudo reboot
```
- `history` shows previously executed commands.
- `shutdown` turns off the system.
- `reboot` restarts the system.

---

## ✅ Part 3 Conclusion

You now master the main commands to get system information, manage processes, check resource usage and control shutdown. All of this is part of Linux administration routine.

In the next part, we'll explore network commands like `ping`, `ssh`, `scp`, `curl` and others.

Let's go! 🌐

[...]

## ✅ Part 3 Conclusion

You now master the main commands to get system information, manage processes, check resource usage and control shutdown. All of this is part of Linux administration routine.

In the next part, we'll explore network commands like `ping`, `ssh`, `scp`, `curl` and others.

Let's go! 🌐

---

## 🌐 Part 4 – Network Commands (Step by Step)

### 🌍 Step 1: Test connectivity with `ping`
```bash
ping google.com
```
- `ping` sends ICMP packets to test if the host is accessible and measures response time.
- Use `Ctrl+C` to interrupt the test.

### 🧾 Step 2: View network interface information
```bash
ip addr
ip route
```
- `ip addr` shows IPs assigned to network interfaces.
- `ip route` displays the machine's routing table.

### 📡 Step 3: View open ports and connections
```bash
netstat -tulnp
```
- `netstat -tulnp` shows open TCP and UDP ports and associated processes.

> Tip: you may need to install the `net-tools` package to use `netstat`.

### 🔐 Step 4: Connect to a remote machine with `ssh`
```bash
ssh usuario@192.168.1.10
```
- `ssh` establishes a secure connection with another computer on the network.

> Replace `usuario` and `IP` with your desired target.

### 📤 Step 5: Send and receive files with `scp`
```bash
scp arquivo.txt usuario@192.168.1.10:/home/usuario/
scp usuario@192.168.1.10:/home/usuario/arquivo.txt ./
```
- `scp` sends or receives files securely via SSH.
- The first example sends, the second downloads.

### 🌐 Step 6: Download files with `wget` and `curl`
```bash
wget https://example.com/arquivo.zip
curl -O https://example.com/arquivo.zip
```
- `wget` and `curl -O` download files directly from the internet.
- `curl` can also be used to access APIs or test endpoints:
```bash
curl https://api.github.com
```

---

## ✅ Part 4 Conclusion

With these network commands, you can test connections, access machines remotely, transfer files and download resources from the internet. This is essential for any administrator, developer or Linux systems enthusiast.

In the next part, we'll talk about permissions, file ownership and terminal security.

See you there! 🔐

---

## 🔐 Part 5 – File Permissions and Properties (Step by Step)

### 🛠️ Step 1: Create test files
```bash
cd ~/lab-linux/arquivos
touch permissao.txt
```
- `touch` creates a new empty file called `permissao.txt`.

### 🔍 Step 2: View current file permissions
```bash
ls -l permissao.txt
```
- `ls -l` shows file permissions, owner and group.

### ✏️ Step 3: Change permissions with `chmod`
```bash
chmod 755 permissao.txt
ls -l permissao.txt
chmod u-x permissao.txt
ls -l permissao.txt
```
- `chmod` sets numeric permissions (e.g. 755 = read/write/execute).
- `u-x` removes execute permission from user.

### 👤 Step 4: View current user and create a new one (optional)
```bash
whoami
# sudo adduser testeusuario  # if you want to create a user for testing
```
- `whoami` shows your current username.
- `adduser` creates a new user on the system (requires sudo).

### 👑 Step 5: Change file owner with `chown`
```bash
sudo chown $USER permissao.txt
```
- `chown` changes the file owner to the current user (`$USER` variable).

### 👥 Step 6: Change file group with `chgrp`
```bash
sudo chgrp $(id -gn) permissao.txt
```
- `chgrp` changes the file's owner group.
- `id -gn` returns the current user's group name.

### 🔁 Step 7: Combine `chown` with group
```bash
sudo chown $USER:$USER permissao.txt
```
- This changes owner and group all at once.

---

## ✅ Part 5 Conclusion

In this part you learned to handle basic Linux security, controlling who can read, write or execute files. This is fundamental to protect your system and properly configure scripts and applications.

In the next part, we'll learn about the most used package managers in the main Linux distributions like Ubuntu, CentOS and Fedora.

Let's go! 📦

---

## 📦 Part 6 – Package Management (Step by Step)

### 📦 Step 1: Update package list on Ubuntu/Debian (`apt`)
```bash
sudo apt update
```
- `apt update` queries repositories and updates the list of available packages.

### 🧰 Step 2: Install a package with `apt`
```bash
sudo apt install cowsay
```
- `apt install` downloads and installs a package. Fun example with `cowsay`.

### ❌ Step 3: Remove a package with `apt`
```bash
sudo apt remove cowsay
```
- `apt remove` uninstalls the package, but may leave configuration files.

> Tip: use `sudo apt purge` to remove everything.

### 📦 Step 4: Update packages on CentOS/RHEL (`yum`)
```bash
sudo yum update
```
- `yum update` updates all system packages.

### 🧰 Step 5: Install package with `yum`
```bash
sudo yum install httpd
```
- `yum install` installs a package on CentOS or RHEL.

### ❌ Step 6: Remove package with `yum`
```bash
sudo yum remove httpd
```
- `yum remove` uninstalls the specified package.

### 📦 Step 7: Use `dnf` (Fedora and newer CentOS versions)
```bash
sudo dnf update
sudo dnf install nano
sudo dnf remove nano
```
- `dnf` is the modern replacement for `yum`. Similar usage, but faster and safer.

> All these commands require administrator permissions (sudo).

---

## ✅ Part 6 Conclusion

You now know the main package managers in the most popular Linux distributions. Knowing how to install, update and remove packages is essential to keep your system functional and secure.

In the next part, we'll see useful and extra commands like `man`, `alias`, and terminal best practices.

Let's finish with style! 🧠

...

## 🧠 Part 7 – Extra Commands and Terminal Best Practices (Step by Step)

### 📚 Step 1: Get help with `man`
```bash
man ls
```
- `man` shows the command manual, with all available options.
- Use `q` to exit the manual.

### 🧩 Step 2: Create shortcuts with `alias`
```bash
alias ll='ls -l'
ll
```
- `alias` allows creating custom commands for easier use.
- `ll` now executes `ls -l`.

> Note: to make permanent, add to `~/.bashrc` or `~/.zshrc`

### 📜 Step 3: View command history
```bash
history | tail -n 5
```
- `history` shows the last executed commands.
- `| tail -n 5` displays only the 5 most recent.

### 🔐 Step 4: Execute commands as administrator with `sudo`
```bash
sudo whoami
```
- `sudo` executes commands as root (administrator user).
- `whoami` returns the current user name.

### ⌨️ Step 5: Use autocomplete with Tab
- Type part of a command or file name and press `Tab` to complete automatically.
```bash
cd ~/lab[TAB]       # completes lab-linux
```

### ➡️ Step 6: Redirect command output
```bash
ls -l > listagem.txt
cat listagem.txt
```
- `>` redirects output to a file (overwrites).
- `>>` adds to the end of the file (without erasing previous content).

### 🔗 Step 7: Use pipes to combine commands
```bash
ps aux | grep bash
```
- The `|` symbol sends the output of one command to another command.
- Here we use `ps aux` and pass the result to `grep bash`.

---

## ✅ Part 7 Conclusion

With these extra commands and practices, you'll use the terminal with more agility, security and organization. Now you have a solid foundation to master Linux in daily use!