# aws-ec2-node-ssl-setup
A walkthrough of EC2 instance launch, node install, nginx and certbot with cron renewal config

## Launch EC2 Instance on AWS

#### 1. Launch new linux ubunutu distribution instance

#### 2. Config security gropu to allow traffic as so:

| PORT  | Type        | Source      |
|-------|-------------|-------------|
| 22    | SSH         | My IP       |
| 80    | HTTP        | 0.0.0.0/0   |
| 443   | HTTPS       | 0.0.0.0/0   |
| 3000  | Custom TCP  | 0.0.0.0/0   |
   
      \* Port 3000 will be used only for testing purposes and must be removed in production.

#### 3. Download the key.pem key and move it to a safe directory
  * Give the key a meaningful name so that you can easily associate it with a specific instance
  * Choose a standard directory for your key so it doesn't get lost like: ```C:\\.ssh```
  * If on a Windows machine use GitBash or WSL terminal - all following steps must be run on a unix terminal
  * For use on WSL, optionally copy the key.pem key from Windows fs to Linux fs as so:
  
```bash
cp /mnt/c/.ssh/key.pem ~/.ssh/
```

#### 4. Change permissions on the key.pem

```bash
sudo chmod 400 ~/.ssh/key.pem
```

#### 5. Test connection to instance

From directory containing key `~/.ssh` connect to instance usinng SSH Connect command from your EC2 Dashboard. If succesfull you will see the command prompt change from local to instance
  
#### 6. This key can be used to access the instance via Filezilla or similar to visualize file structure
For use with Filezilla:
  * <kbd>Edit</kbd> &rarr; <kbd>Setting</kbd> &rarr; <kbd>SFTP</kbd> &rarr; locate you key and press <kbd>Add Key</kbd>
  * <kbd>File</kbd> &rarr; <kbd>Site Manager</kbd> &rarr; if not already created, <kbd>New Site</kbd> &rarr; click on the <kbd>yoursite.com</kbd>.
  * In <kbd>General</kbd> tab set:
      - Protocol: **SFTP**
      - Host: your instance **IPv4**
      - Port: can be set to **22** but not necessarely
      - Logon Type: **Key file**
      - User: **ubunut** (default for EC2 instances)
      - Key file: **C:\\.ssh\key.pem**
   
#### 7. Once logged onto your instance update and ugrade the system
\* Note: From now on all bash commands are executed in the instance teriminal. Keep an eye on the terminal prompt as you can get automatically logged out after inactivity.

```bash
sudo apt update
```

```bash
sudo apt upgrade
```

#### 8. Install Node.js via NVM
 



