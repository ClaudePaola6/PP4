# PP4

## Goal

In this exercise you will:

* Use SSH to connect to remote servers from WSL, macOS, or Linux shells, understanding the handshake and authentication process.
* Generate an Ed25519 SSH key pair and explain the concept of digital signatures.
* Configure your local SSH client via the `~/.ssh/config` file for streamlined access.
* Securely copy files between local and remote hosts using `scp`, including local-to-remote, remote-to-local, and remote-to-remote transfers.
* Automate startup tasks on the remote server by writing a shell script that runs at login and explaining the role of `~/.bashrc` vs. `~/.profile`.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. Once time is up, stop immediately and record exactly where you paused.

---

## Workflow

1. **Fork** this repository
2. **Modify & commit** your solution
3. **Submit your link for Review**

---

## Prerequisites

* Several starter repos are available here:
  [https://github.com/orgs/STEMgraph/repositories?q=SSH%3A](https://github.com/orgs/STEMgraph/repositories?q=SSH%3A)
* Consult the SSH and SCP man-pages for detailed options and explanations:

  * `man ssh`
  * `man scp`

---

## Tasks

### Task 1: SSH Login

**Objective:** Establish an SSH connection and observe each stage of the process.

1. From your local shell (WSL, macOS Terminal, or Linux), log into the `vorlesungsserver` (or any other remote machine of your choice, e.g. your own raspberry pi):

   ```bash
   ssh youruser@remotehost
   ```
2. Carefully observe and note each step:

   * **TCP connection** to port 22 on `remotehost`.
   * **SSH protocol handshake**: key exchange and algorithm negotiation.
   * **Authentication**: public-key or password exchange.
   * **Shell allocation**: your remote session starts.
3. After login, exit the session with `exit`.

**Provide:**

```bash
# 1) The exact ssh command you ran
# 2) A detailed, step-by-step explanation of what happened at each stage

 Detaillierte Erklärung des Anmeldevorgangs.
	1.	TCP-Verbindung :
	- Dein Terminal baut eine TCP-Verbindung zu Port 22 des Remotehosts (IP-Adresse) auf.
	- Das bedeutet, dass dein Computer eine Anfrage an den SSH-Standardport des entfernten Hosts sendet.
	2.	Handshake des SSH-Protokolls :
	- Aushandlung der Algorithmen (Verschlüsselung, Integrität, Komprimierung).
	- Schlüsselaustausch: in der Regel über Diffie-Hellman oder elliptische Kurven.
	- Client und Server einigen sich auf temporäre Sitzungsschlüssel.
	3.	Authentifizierung:
	- Der Client präsentiert entweder ein Passwort oder einen öffentlichen Schlüssel.
	- Wenn es sich um einen öffentlichen Schlüssel handelt, beweist der Client, dass er den entsprechenden privaten Schlüssel besitzt, indem er eine Challenge unterschreibt.
	4.	Zuteilung einer Shell :
	- Nachdem du dich authentifiziert hast, startet der Server eine Remote-Shell (bash, zsh usw.).
	- Du kannst nun mit dem entfernten Rechner interagieren.
	5.	Abmelden:
	- Wenn du exit eingibst, wird die Shell geschlossen und die SSH-Sitzung ist beendet.

### Task 2: Ed25519 Key Pair

**Objective:** Create a secure key pair and explain how digital signatures verify identity.

1. Generate an Ed25519 SSH key pair:

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   * Accept the default file location (`~/.ssh/id_ed25519`). Or provide the `-f <filepath>` option additionally.
   * Enter a passphrase when prompted (optional).
2. Locate and inspect your `id_ed25519` (private key) and `id_ed25519.pub` (public key).
3. Install your key on the remote machine (e.g. `vorlesungsserver`.
4. Explain in writing:

   * How the **private key** is used to sign challenges.
   * How the **public key** on the server verifies signatures without revealing the private key.
   * Why Ed25519 is preferred (performance, security).

**Provide:**

```bash
# 1) The ssh-keygen command you ran
# 2) The file paths of the generated keys
# 3) Your written explanation (3–5 sentences) of the signature process

2) Speicherort der erzeugten Dateien.
	- Privater Schlüssel: ~/.ssh/id_ed25519
	- Öffentlicher Schlüssel: ~/.ssh/id_ed25519.pub

3) Erklärung, wie digitale Signaturen funktionieren.
	- Bei einer SSH-Verbindung sendet der Server eine Challenge an den Client.
	- Der Client signiert diese Challenge mit seinem privaten Schlüssel Ed25519.
	- Der Server besitzt den entsprechenden öffentlichen Schlüssel (in ~/.ssh/authorized_keys) und kann die Signatur überprüfen, ohne jemals den privaten Schlüssel zu sehen.
	- Ed25519 wird aufgrund seiner Post-Quantum-Sicherheit, seiner Geschwindigkeit und seiner geringen Ressourcenkosten bevorzugt.



### Task 3: SSH Config File

**Objective:** Simplify SSH commands via `~/.ssh/config`.

1. Open (or create) `~/.ssh/config` in `vim`.
2. Add entries for your hosts, for example:

   ```text
   Host my-remote
       HostName remote.example.com
       User youruser
       IdentityFile ~/.ssh/id_ed25519

   Host backup-server
       HostName backup.example.com
       User backupuser
       Port 2222
       IdentityFile ~/.ssh/id_ed25519_backup
   ```
3. Save and close the file, then test:

   ```bash
   ssh my-remote
   ssh backup-server
   ```
4. Explain:

   * How SSH reads `~/.ssh/config` and matches hosts.
   * The difference between `HostName` and `Host`.
   * How aliases prevent long commands.

**Provide:**

```text
# 1) The full contents of your ~/.ssh/config
# 2) A short explanation (3–4 sentences) of how the config simplifies connections

Die Datei ~/.ssh/config ermöglicht es, SSH-Verbindungen mit Aliasnamen zu vereinfachen.
	- Host ist der benutzerdefinierte Alias, den du verwenden kannst (z. B. ssh my-remote).
	- HostName ist die tatsächliche Adresse des Rechners.
	- Das erspart es dir, jedes Mal den ganzen Befehl mit -i, -p und dem Benutzernamen neu eingeben zu müssen.


### Task 4: SCP File Transfers

**Objective:** Practice copying files securely using `scp`.

1. **Local → Remote**:

   ```bash
   scp /path/to/localfile.txt youruser@remotehost:~/destination/
   ```
2. **Remote → Local**:

   ```bash
   scp youruser@remotehost:~/remotefile.log ./local_destination/
   ```
3. **Remote → Remote** (between two directories on the same remote host):

   ```bash
   scp -r youruser@remotehost:/path/dir1 youruser@remotehost:/path/dir2
   ```
4. For each command:

   * Verify file timestamps and sizes after transfer, using `ls -la`
   * Note any flags you used (e.g., `-r`, `-P` for port).
5. Explain:

   * How `scp` initiates an SSH session for each transfer.
   * The role of encryption in protecting data in transit.

**Provide:**

```bash
# 1) Each scp command you ran
# 2) Any flags or options used
# 3) A brief explanation (2–3 sentences) of scp’s mechanism


Die Datei ~/.ssh/config ermöglicht es, SSH-Verbindungen mit Aliasnamen zu vereinfachen.
	- Host ist der benutzerdefinierte Alias, den du verwenden kannst (z. B. ssh my-remote).
	- HostName ist die tatsächliche Adresse des Rechners.
	- Das erspart es dir, jedes Mal den ganzen Befehl mit -i, -p und dem Benutzernamen neu eingeben zu müssen.

### Task 5: Login Shell Script & Profile Explanation

**Objective:** Automate commands at login and understand shell initialization files.

1. On the **remote** server, create a script `~/login_tasks.sh` containing at least three commands you find useful (e.g., `echo "Welcome $(whoami)"`, `uptime`, `ls ~/projects`). You may either use `vim` or try the following to create a file from your commandline directely:

   ```bash
   cat << 'EOF' > ~/login_tasks.sh
   #!/usr/bin/env bash
   echo "Welcome $(whoami)! Today is $(date)."
   uptime
   ls ~/projects
   EOF
   chmod +x ~/login_tasks.sh
   ```

> The files content should be something akin to:
> ```bash
> #!/usr/bin/env bash
> echo "Welcome $(whoami)! Today is $(date)."
> uptime
> ls ~/projects
> ```

2. Append to your `~/.bashrc` (or `~/.profile` if using a login shell) a line to source this script on each new session:

   ```bash
   echo "source ~/login_tasks.sh" >> ~/.bashrc
   ```
3. Log out and log back in to trigger the script.
4. Explain:

   * The difference between `~/.bashrc` and `~/.profile` (interactive vs. login shells).
   * Why and when each file is read.
   * How sourcing differs from executing.

**Provide:**

```bash
# 1) The contents of login_tasks.sh
# 2) The lines you added to ~/.bashrc or ~/.profile
# 3) Your explanation (3–5 sentences) of shell init files and sourcing vs. executing
```
- ~/.bashrc wird bei jeder interaktiven Nicht-Login-Shell (z. B. neues Terminal) ausgeführt.
	- ~/.profile wird für alle Login-Shells ausgeführt (Anmeldung über SSH, oder Konsole).
	- source führt das Skript im aktuellen Kontext der Shell aus, was den Zugriff auf Variablen oder Änderungen ermöglicht.
	- ./script.sh würde in einem neuen Bash-Prozess ausgeführt.
---

**Remember:** Stop working after **90 minutes** and record where you stopped.
