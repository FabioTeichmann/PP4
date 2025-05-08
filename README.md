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
```
1)  ssh fabioteichmann@flipbook
2) -Es wurde eine TCP-Verbindung zu Port 22 von flipbook aufgebaut.
   -Der SSH-Client und der Server führten einen Handshake durch (Aushandlung von         
    Verschlüsselung und Austausch temporärer Schlüssel).
   -Die Authentifizierung erfolgte per Passwort.
   -Nach erfolgreicher Anmeldung wurde eine Bash-Shell auf dem Remote-Server geöffnet.
   -Mit "exit" wurde die Verbindung korrekt beendet.
---

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
```
1) ssh-keygen -t ed25519 -C "fabio.teichmann@stud-thga.de"
2) Enter file in which to save the key (/home/fabioteichmann/.ssh/id_ed25519)
3) Beim Login über SSH sendet der Server eine zufällige Prüfnachricht an meinen lokalen Rechner. Mein SSH-Client verwendet den privaten Schlüssel "~/.ssh/id_ed25519", um diese Nachricht mit meiner Passphrase zu signieren. Der Server überprüft diese Signatur mithilfe des öffentlichen Schlüssels, der in "~/.ssh/authorized_keys" hinterlegt ist. Ist die Signatur gültig, wird der Zugriff gewährt ohne dass ein Passwort für das Benutzerkonto nötig ist. Der private Schlüssel bleibt dabei stets sicher auf meinem Gerät gespeichert.


---

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
  
   1)Wenn man ein SSH-Befehl wie ssh server1 eingibt, durchsucht SSH die Datei ~/.ssh/config nach einem passenden Host-Eintrag. Sobald ein passender Eintrag gefunden wird, übernimmt SSH die dort definierten Einstellungen wie HostName, User, Port und IdentityFile. Dadurch werden die Verbindungsdetails automatisch ergänzt, ohne dass man sie jedes mal manuell angeben musst.
   2)Host ist ein Alias, den man selbst wählt und später beim Verbindungsaufbau verwendet – z. B. ssh Server1.
HostName dagegen ist der echte Name oder die IP-Adresse des Servers, zu dem die Verbindung hergestellt wird – also z. B. server1.example.com 
Kurz: Host = ein Spitzname für den Server, HostName = die tatsächliche Adresse.
  3)anstatt ssh -i ~/.ssh/id_ed25519 -p 2222 fabioteichmann@server2.example.com
einzugeben, kann man einfach schreiben: ssh Server2
Das spart Zeit, vermeidet Tippfehler und macht das Arbeiten mit mehreren Servern übersichtlich und effizient.

**Provide:**

```text
# 1) The full contents of your ~/.ssh/config
# 2) A short explanation (3–4 sentences) of how the config simplifies connections
```
1)
Host Server1
        HostName flipbook
        User fabioteichmann
        IdentityFile ~/.ssh/id_ed25519

Host Server2
        Hostname flipbook
        User fabioteichmann
        IdentityFile ~/.ssh/id_ed25519_Server2

2) Die Datei "~/.ssh/config" speichert Verbindungsdetails SSH-Ziele. Durch Aliase wie Server1 und Server2 kann man sich mit einem kurzen Befehl verbinden, ohne jedes Mal Hostname, Benutzername, Port und Schlüsselpfad manuell anzugeben. 
---

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
```
1)  Datei von lokal → remote
"scp ~/Dokumente/bericht.txt Server1:~/uploads/"
    Datei von remote → lokal
"scp Server1:~/uploads/bericht.txt ~/Downloads/"
Datei von einem Verzeichnis zu einem anderen auf dem gleichen Remote-Host (Server1)
"scp -r Server1:~/uploads/ Server1:~/backup/"

2) "-r" wird verwendet, um ganze Verzeichnisse samt Inhalt zu kopieren.
   Hostalias „Server1“ wurde über die Datei ~/.ssh/config definiert, wodurch HostName, User und IdentityFile automatisch übernommen wurden

3) scp überträgt Dateien über eine verschlüsselte SSH-Verbindung. Dabei erfolgt die Authentifizierung wie bei einer normalen SSH-Sitzung per Passwort. Die gesamte Übertragung ist gegen Mitlesen und Manipulation geschützt.

---

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

---

**Remember:** Stop working after **90 minutes** and record where you stopped.
