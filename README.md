# Jenkins CI/CD Project — Setup Report

This report documents each step taken to provision an AWS EC2 instance and install Jenkins. All actions were performed manually from the AWS Console and a local terminal. This file will be updated as the project continues.

---

## Phase 1 — EC2 Instance Setup

### Step 1 — AWS Console Was Opened

The AWS Management Console was opened and the **EC2** dashboard was navigated to. The **Europe (Stockholm) — eu-north-1** region was selected.

---

### Step 2 — A New Instance Was Launched

The **Launch instance** button was clicked to begin creating a new virtual server.

---

### Step 3 — The Instance Was Named

The instance was named **`ci-jenkins-controller`**.

---

### Step 4 — The Operating System Was Selected

**Ubuntu Server 24.04 LTS (64-bit x86)** was selected as the Amazon Machine Image (AMI).

---

### Step 5 — The Instance Type Was Chosen

The instance type **`t3.medium`** was selected to provide sufficient resources for running Jenkins.

---

### Step 6 — A Key Pair Was Created

A new key pair was created with the following settings:

| Setting | Value |
|---|---|
| Key pair name | `ci-jenkins-key` |
| Key pair type | RSA |
| File format | `.pem` |

The private key file was downloaded and saved to a secure local folder.

---

### Step 7 — Network and Security Were Configured

A security group was created with the following settings:

| Setting | Value |
|---|---|
| Security group name | `ci-jenkins-sg` |
| Description | Security group for Jenkins CI controller access |

The following inbound rules were added:

| Port | Protocol | Source | Purpose |
|---|---|---|---|
| 22 | SSH | My IP | Server access |
| 8080 | Custom TCP | My IP | Jenkins web UI |

---

### Step 8 — Storage Was Configured

The root volume was configured with **20 GiB** of **gp3** storage.

---

### Step 9 — The Instance Was Launched

The **Launch instance** button was clicked to create the server.

---

### Step 10 — The Instance Status Was Verified

The EC2 **Instances** page was opened to confirm the instance was in a **Running** state with **2/2 status checks passed**.

![EC2 instance running](screenshots/10-ec2-instance-running.png)

---

## Phase 2 — Server Connection and Jenkins Installation

### Step 11 — The Running Instance Was Selected

The running **`ci-jenkins-controller`** instance was selected from the EC2 instances list.

---

### Step 12 — Connection Details Were Opened

The **Connect** button was clicked and the **SSH client** tab was opened to view the connection instructions.

![EC2 SSH connect page](screenshots/11-ec2-ssh-connect.png)

---

### Step 13 — The SSH Command Was Copied

The example SSH command was copied from the AWS connection page:

```bash
ssh -i "ci-jenkins-key.pem" ubuntu@ec2-51-21-149-38.eu-north-1.compute.amazonaws.com
```

---

### Step 14 — The Local Terminal Was Opened

A terminal was opened on the local machine and the directory containing the `.pem` key file was navigated to:

```bash
cd /path/to/key-directory
ls
```

The presence of `ci-jenkins-key.pem` was confirmed.

---

### Step 15 — Key Permissions Were Set

The private key file permissions were restricted:

```bash
chmod 400 ci-jenkins-key.pem
```

---

### Step 16 — A Connection to the Server Was Established

The SSH connection to the EC2 instance was made:

```bash
ssh -i "ci-jenkins-key.pem" ubuntu@51.21.149.38
```

The host fingerprint was accepted on first connection. A successful login was confirmed.

---

### Step 17 — The Server Was Updated

The package lists and installed packages were updated:

```bash
sudo apt update && sudo apt upgrade -y
```

---

### Step 18 — Java Was Installed

OpenJDK was installed as a prerequisite for Jenkins:

```bash
sudo apt install fontconfig openjdk-17-jre -y
sudo apt install openjdk-21-jre -y
```

OpenJDK 21 was required by Jenkins LTS 2.555.x, so both versions were installed on the server.

---

### Step 19 — The Java Installation Was Verified

The installed Java version was checked:

```bash
java -version
```

OpenJDK 21 was confirmed as the active version.

---

### Step 20 — The Official Jenkins Documentation Was Consulted

The [Jenkins Linux installation guide](https://www.jenkins.io/doc/book/installing/linux/) was reviewed for the correct repository and package installation commands.

![Jenkins official installation documentation](screenshots/17-jenkins-official-docs.png)

---

### Step 21 — The Jenkins Repository Key Was Added

The Jenkins GPG signing key was downloaded:

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

The updated `jenkins.io-2026.key` was used because the older 2023 key had expired.

---

### Step 22 — The Jenkins Repository Was Added

The Jenkins APT repository was added to the system:

```bash
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

The package lists were refreshed:

```bash
sudo apt update
```

---

### Step 23 — Jenkins Was Installed

Jenkins was installed from the configured repository:

```bash
sudo apt install jenkins -y
```

---

### Step 24 — Jenkins Was Configured and Started

Jenkins was configured to use Java 21, enabled on boot, and started:

```bash
echo "JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64" | sudo tee /etc/default/jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

---

### Step 25 — The Jenkins Service Status Was Checked

The running state of the Jenkins service was verified:

```bash
sudo systemctl status jenkins
```

The output showed **`Active: active (running)`**.

---

### Step 26 — The Initial Admin Password Was Retrieved

The one-time administrator password was retrieved from the server:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

The password was copied for use in the setup wizard.

---

### Step 27 — Jenkins Was Accessed in the Browser

The Jenkins web interface was opened at:

```
http://51.21.149.38:8080
```

The **Unlock Jenkins** page was displayed. The initial admin password was entered and **Continue** was clicked.

![Jenkins unlock page](screenshots/26-jenkins-unlock.png)

---

### Step 28 — Suggested Plugins Were Installed

**Install suggested plugins** was selected on the setup wizard. The recommended plugin set was downloaded and installed automatically.

![Jenkins plugin installation](screenshots/27-jenkins-plugins-install.png)

---

### Step 29 — An Administrator User Was Created

On the **Create First Admin User** page, an administrator account was created with a username and password. **Save and Continue** was clicked.

---

### Step 30 — The Jenkins URL Was Configured

On the **Instance Configuration** page, the Jenkins URL was confirmed as:

```
http://51.21.149.38:8080/
```

**Save and Finish** was clicked.

![Jenkins instance configuration](screenshots/28-jenkins-instance-config.png)

---

### Step 31 — The Jenkins Dashboard Was Reached

The Jenkins dashboard was displayed, confirming that the installation and initial setup were completed successfully. Jenkins **2.555.2** was running and ready for use.

![Jenkins dashboard](screenshots/29-jenkins-dashboard.png)

---

*This report will be extended as further CI/CD configuration steps are completed.*
