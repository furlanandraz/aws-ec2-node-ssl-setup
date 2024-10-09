# aws-ec2-node-ssl-setup
A walkthrough of EC2 instance launch, node install, nginx and certbot with cron renewal config. **Linux ubuntu distribution specific.**

## Launch EC2 Instance on AWS

#### 1. Launch new linux ubunutu distribution instance

#### 2. Config security gropu to allow traffic as so:

| PORT  | Type        | Source      |
|-------|-------------|-------------|
| 22    | SSH         | My IP       |
| 80    | HTTP        | 0.0.0.0/0   |
| 443   | HTTPS       | 0.0.0.0/0   |
| 3000  | Custom TCP  | 0.0.0.0/0   |
   
\* Port 3000 will be used only for testing purposes and must be removed in production. Adjust port to your app.

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

#### 8. Install Node.js via NVM (Node Version Manager)

* Navigate to [Node Package Manager](https://nodejs.org/en/download/package-manager)
* Select: Install Node.js on <kbd>LTS</kbd> on <kbd>Linux</kbd> using <kbd>nvm</kbd>
* Follow the bash commands provided

\* NVM allows for Node.js version control.

#### 9. Connect instance to github via SSH

\* Assuming there is a github repository with your Node.js project that you want to run.
* Create a new public key on your instance (rsa or other guthub supported key type) and confirm the default config that is prompted:

```bash
ssh key gen -t rsa
```

* This command outputs the key in `~/.ssh/`
* Open the key file and copy the record from the key to your **clipboard** (key name my differ based on key type you generted)

```bash
cat ~/.ssh/id_rsa.pub
```

* On github go to <kbd>Profile</kbd> &rarr; <kbd>Settings</kbd> &rarr; <kbd>SSH and GPG keys</kbd> &rarr; <kbd>Add New SSH key</kbd>
   - Key type: Authentication Key
   - Key: paste from **clipboard**
 
#### 10. Test connection to github

\*Make sure to run this command in your `~/.ssh/folder`

```bash
ssh git@github
```

#### 11. Clone your Node.js project

* In you repo on github, go to <kbd>Code</kbd> &rarr;  <kbd>SSH</kbd>  and copy the command <git_url>
* Inside `/home/ubuntu/` run the SSH command with code snippet you copied

```bash
git clone <git_url>
```

* This command will create a directory and clone your repo - no need to make a new directory
* Navigate to the newly created directory `/home/ubunut/your_app/` and run dependency install

```bash
npm install
```

* Optionally start your app which will now be accessible via previously opened port e.g. 3000

```bash
npm run start
```

* Navigate to your instance IP and search for your port e.g. `IP:3000` or run a curl command

```bash
curl IP:3000
```
* If this fails check your app on localhost:

```bash
curl localhost:3000
```

#### 12. Enironment file
This file is used to securely store any sensitive data if it exists

* name this file so it reflects your project, or simply `app.env`

```bash
sudo vim /etc/app.env
```
* Paste your variables in the env file
* Change permission and ownership of the file

```bash
sudo chmod 600 /etc/app.env
```

```bash
sudo chown ubuntu:ubuntu /etc/app.env
```

#### 13. Systemd service file
This file keeps your app runing and restarts it at reboot
* In this scenario `npm` scripts are not accessible, so run your enty point with `node` command - this is due to using nvm
* to find your node path for ExecStart, run

```bash
which node
```
* If you want to use dedicated logs, create the files, or set the outputs to the standard `syslog`

```bash
sudo mkdir /var/log/app/
```
```bash
sudo touch /var/log/app/output.log /var/log/app/output.log
```
```bash
sudo vim /etc/systemd/system/app.service
```
```text
[Unit]
Description=Node.js App
After=network.target multi-user.target

User=ubuntu
WorkingDirectory=/home/ubuntu/your_app
EnvironmentFile=/etc/app.env
ExecStart=/home/ubuntu/.nvm/versions/node/v20.18.0/bin/node index.js
Environment=NODE_ENV=production
StandardOutput=file:/var/log/app/output.log
StandardError=file:/var/log/app/error.log
SyslogIdentifier=your_app_name
Restart=on-failure

[Install]
WantedBy=multi-user.target
````

#### 13. Run the systemd service file

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl enable app.service
```

```bash
sudo systemctl start app.service
```

* Check status of systemd

```bash
sudo journalctl -n app.service
```

