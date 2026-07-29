---
title: 'The Secure Shell Service'
description: 'OpenSSH, system access, file transfer'
---
## The OpenSSH Service 

**Secure Shell (SSH)** 
- Delivers a secure mechanism for data transmission between source and destination systems over IP networks. 
- Designed to replace the old remote login programs that transmitted user passwords in clear text and data unencrypted. 
- Employs digital signatures for user authentication with encryption to secure a communication channel. 
	- this makes it extremely hard for unauthorized people to gain access to passwords or the data in transit. 
- Monitors the data being transferred throughout a session to ensure integrity. 
- Includes a set of utilities `ssh` and `sftp` for remote users to log in, transfer files, and execute commands securely.

### Common Encryption Techniques

- Two common techniques: symmetric and asymetric

**Symmetric Technique**

- Secret key encryption.
- Uses a single key called a secret key that is generated as a result of a negotiation process between two entities at the time of their initial contact. 
- Both sides use the same secret key during subsequent communication for data encryption and decryption.

**Asymmetric Technique**

- Public key encryption
- Combination of private and public keys
	- Randomly generated and mathematically related strings of alphanumeric characters 
	- attached to messages being exchanged. 
- The client transmutes the information with a public key and the server decrypts it with the paired private key.
- Private key must be kept secure since it is private to a single sender
- the public key is disseminated to clients.
- used for channel encryption and user authentication.

### Authentication Methods 

- Encrypted channel is established between the client and server
- Then additional negotiations take place between the two to authenticate the user trying to access the server. 
- Methods listed in the order in which they are attempted during the authentication process:

1. GSSAPI-based ( Generic Security Service Application Program Interface) authentication
2. Host-based authentication
3. Public key-based authentication
4. Challenge-response authentication
5. Password-based authentication

**GSSAPI-Based Authentication** 

- Provides a standard interface that allows security mechanisms, such as Kerberos, to be plugged in. 
- OpenSSH uses this interface and the underlying Kerberos for authentication. 
- Exchange of tokens takes place between the client and server to validate user identity.

**Host-Based Authentication** 

- Allows a single user, a group of users, or all users on the client to be authenticated on the server. 
- A user may be configured to log in with a matching username on the server or as a different user that already exists there. 
- For each user that requires an automatic entry on the server, a \~/.shosts file is set up containing the client name or IP address, and, optionally, a different username.
- The same rule applies to a group of users or all users on the client that require access to the server. 
	- In that case, the setup is done in the /etc/ssh/shosts.equiv file on the server.

**Private/Public Key-Based Authentication** 

- Uses a private/public key combination for user authentication.
- User on the client has a private key and the server stores the corresponding public key. 
- At the login attempt, the server prompts the user to enter the passphrase associated with the key and logs the user in if the passphrase and key are validated.

**Challenge-Response Authentication** 

- Based on the response(s) to one or more arbitrary challenge questions that the user has to answer correctly in order to be allowed to log in to the server.

**Password-Based Authentication** 

- Last fall back option. 
- Server prompts the user to enter their password. 
- Checks the password against the stored entry in the shadow file and allows the user in if the password is confirmed.

### OpenSSH Protocol Version and Algorithms 

- V2
- Supports various algorithms for data encryption and user authentication (digital signatures) such as:

**RSA** (Rivest-Shamir-Adleman)
- More prevalent than the rest
- Supports both encryption and authentication.

**DSA and ECDSA** (Digital Signature Algorithm and Elliptic Curve Digital Signature Algorithm)
- Authentication only.
- Used to generate public and private key pairs for the asymmetric technique.

### OpenSSH Packages 

- Installed during OS installation

**openssh**
- provides the `ssh-keygen` command and some library routines

**openssh-clients**
- includes commands, such as `sftp`, `ssh`, and `ssh-copy-id`, and a client configuration file */etc/ssh/ssh_config*

**openssh-server**
-  contains the sshd service daemon, server configuration file */etc/ssh/sshd_config*, and library routines.

### OpenSSH Server Daemon and Client Commands 

- OpenSSH server program is sshd

**sshd**
- Preconfigured and operational on new RHEL installations
- Allows remote users to log in to the system using an ssh client program such as PuTTY or the ssh command. 
- Daemon listens on TCP port 22 
	- Documented in the /etc/ssh/sshd_config file with the Port directive.

- Use sftp instead of scp do to scp security flaws. 

`sftp`                              
- Secure remote file transfer program

`ssh`                         
 - Secure remote login command

`ssh-copy-id`                       
  - Copies public key to remote systems

`ssh-keygen`                          
  - Generates and manages private and public key pairs

### Server Configuration File 

**/etc/ssh/sshd_config**

**/var/log/secure** 
- log file is used to capture authentication messages.

View directives listed in */etc/ssh/sshd_config*:
```bash
[root@server30 tmp]# cat /etc/ssh/sshd_config
```


**Port**                                
  - Port number to listen on. Default is 22.

**Protocol**                            
  - Default protocol version to use.

**ListenAddress**                       
  - Sets the local addresses the sshd service should listen on. 
  - Default is to listen on all local addresses.

**SyslogFacility**                      
  - Defines the facility code to be used when logging messages to the /var/log/secure file. This is based on the configuration in the /etc/rsyslog.conf file. Default is **AUTHPRIV.**

**LogLevel**                            
  Identifies the level of criticality for the messages to be logged. Default is **INFO**.

**PermitRootLogin**                     
  Allows or disallows the root user to log in directly to the system. Default is yes.

**PubKeyAuthentication**                
  Enables or disables public key-based authentication. Default is yes.

 **AuthorizedKeysFile**                  
  Sets the name and location of the file containing a user's authorized keys. Default is \~/.ssh/authorized_keys.

**PasswordAuthentication**              
  Enables or disables local password authentication. Default is yes.

**PermitEmptyPasswords**                
  Allows or disallows the use of null passwords. Default is no.

**ChallengeResponseAuthentication**     
  Enables or disables challenge-response authentication mechanism. Default is yes.

**UsePAM**                              
  Enables or disables user authentication via PAM. If enabled, only root will be able to run the sshd daemon. Default is yes.

**X11Forwarding**                       
  Allows or disallows remote access to graphical applications. Default is yes.

### Client Configuration File 

**/etc/ssh/ssh_config**
- Local configuration file that directs how the client should behave. This file, , is located in the /etc/ssh directory.
- Directives preset in this file that affect all outbound ssh communication.

View the default directive settings:
`[root@server30 tmp]# cat /etc/ssh/sshd_config`

**Host**                                
  - Container that declares directives applicable to one host, a group of hosts, or all hosts. 
  - Ends when another occurrence of Host or Match is encountered. Default is \*,  (all hosts)

**ForwardX11**                          
  - Enables or disables automatic redirection of X11 traffic over SSH connections. 
  - Default is no.

**PasswordAuthentication**              
  - Allows or disallows password authentication. 
  - Default is yes.

**StrictHostKeyChecking**               
  - Whether to add host keys (host fingerprints) to  \~/.ssh/known_hosts when accessing a host for the first time
  - What to do when the keys of a previously accessed host mismatch with what is stored in \~/.ssh/known_hosts. 
- no: 
	 - Adds new host keys and ignores changes to existing keys.

- yes: 
	- Adds new host keys and disallows connections to hosts with non-matching keys. 

- accept-new: 
	- Adds new host keys and disallows connections to hosts with non-matching keys. 
	
- ask (default): 
	- Prompts whether to add new host keys and disallows connections to hosts with non-matching keys.

**IdentityFile**                        
  - Defines the name and location of a file that stores a user's private key for their identity validation. 
  - Defaults are:
	- id_rsa, id_dsa, and  id_ecdsa based on the type of algorithm used.
	- Corresponding public key files with .pub extension are also stored at the same directory location.

**Port**                                
  Sets the port number to listen on. Default is 22.

**Protocol**                            
  Specifies the default protocol version to use

\~/.ssh/
- does not exist by default
- created when:
	- a user executes the `ssh-keygen` command for the first time to generate a key pair
	- A user connects to a remote ssh server and accepts its host key for the first time.
		- The client stores the server's host key locally in a file called known_hosts along with its hostname or IP address. 
		- On subsequent access attempts, the client will use this information to verify the server's authenticity.

## System Access and File Transfer 

### Lab: Access RHEL System from Another RHEL System 

- issue the ssh command as user1 on server10 to log in to server20. 
- Run appropriate commands on server20 for validation.
- Log off and return to the originating system.

1\. Issue the ssh command as user1 on server10:

```bash
[user1@server30 tmp]$ ssh server20
```

2\. Issue the basic Linux commands whoami, hostname, and pwd to confirm
that you are logged in as user1 on server20 and placed in the correct
home directory:
```bash
[user1@server40 ~]$ whoami
user1
[user1@server40 ~]$ hostname
server40
[user1@server40 ~]$ pwd
/home/user1

```

3\. Run the logout or the exit command or simply press the key combination Ctrl+d to log off server20 and return to server10:
```bash
[user1@server40 ~]$ exit
logout
Connection to server40 closed.
```



If you wish to log on as a different user such as user2 (assuming user2
exists on the target server server20), you may run the ssh command in
either of the following ways:

`[user1@server30 tmp]$ ssh -l user2 server40`

`[user1@server30 tmp]$ ssh user2@server40`

### Lab: Generate, Distribute, and Use SSH Keys 

- Generate a passwordless ssh key pair using RSA algorithm for user1 on server10.
- display the private and public file contents. 
- Distribute the public key to server20 and attempt to log on to server20 from server10. 
- Show the log file message for the login attempt.

1\. Log on to server10 as user1.

2\. Generate RSA keys without a password (-N) and without detailed
output (-q). Press Enter when prompted to provide the filename to store
the private key.
```bash
[user1@server30 tmp]$ ssh-keygen -N "" -q
Enter file in which to save the key (/home/user1/.ssh/id_rsa): 
```

View the private key:
`[user1@server30 tmp]$ cat ~/.ssh/id_rsa`

View the public key:
`[user1@server30 tmp]$ cat ~/.ssh/id_rsa.pub`

3\. Copy the public key file to server20 under /home/user1/.ssh
directory. 
```bash
user1@server30 tmp]$ ssh-copy-id server40
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user1/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
user1@server40's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'server40'"
and check to make sure that only the key(s) you wanted were added.

```

- This command also creates or updates the known_hosts
file on server10 and stores the fingerprints for server20 in it. 

`[user1@server30 tmp]$ cat ~/.ssh/known_hosts`

4\. On server10, run the ssh command as user1 to connect to server20.
You will not be prompted for a password because there was none assigned
to the ssh keys.
```bash
[user1@server30 tmp]$ ssh server40
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Sun Jul 21 01:20:17 2024 from 192.168.0.30
```

View this login attempt in the /var/log/secure file on server20:
`[user1@server40 ~]$ sudo tail /var/log/secure`

### Executing Commands Remotely Using ssh 

- Can use `ssh` command to run programs without remoting in:

Execute the hostname command on server20:
```bash
[user1@server30 tmp]$ ssh server40 hostname
server40
```


Run the `nmcli` command on server20 to show (s) active network connections(c):
```bash
[user1@server30 tmp]$ ssh server40 nmcli c s
NAME    UUID                                  TYPE      DEVICE 
enp0s3  1c391bb6-20a3-4eb4-b717-1e458877dbe4  ethernet  enp0s3 
lo      175f8a4c-1907-4006-b838-eb43438d847b  loopback  lo 
```

### sftp` command 
- Interactive file transfer tool. 

On server10, to connect to server20:
```bash
[user1@server30 tmp]$ sftp server40
Connected to server40.
sftp> 
```

Type ? at the prompt to list available commands along with a short description:
```bash
[user1@server30 tmp]$ sftp server40
Connected to server40.
sftp> ?
Available commands:
bye                                Quit sftp
cd path                            Change remote directory to 'path'
chgrp [-h] grp path                Change group of file 'path' to 'grp'
chmod [-h] mode path               Change permissions of file 'path' to 'mode'
chown [-h] own path                Change owner of file 'path' to 'own'
df [-hi] [path]                    Display statistics for current directory or
                                   filesystem containing 'path'
exit                               Quit sftp
get [-afpR] remote [local]         Download file
help                               Display this help text
lcd path                           Change local directory to 'path'
lls [ls-options [path]]            Display local directory listing
lmkdir path                        Create local directory
ln [-s] oldpath newpath            Link remote file (-s for symlink)
lpwd                               Print local working directory
ls [-1afhlnrSt] [path]             Display remote directory listing
lumask umask                       Set local umask to 'umask'
mkdir path                         Create remote directory
progress                           Toggle display of progress meter
put [-afpR] local [remote]         Upload file
pwd                                Display remote working directory
quit                               Quit sftp
reget [-fpR] remote [local]        Resume download file
rename oldpath newpath             Rename remote file
reput [-fpR] local [remote]        Resume upload file
rm path                            Delete remote file
rmdir path                         Remove remote directory
symlink oldpath newpath            Symlink remote file
version                            Show SFTP version
!command                           Execute 'command' in local shell
!                                  Escape to local shell
?                                  Synonym for help
```

Example:

```bash
sftp> ls
sftp> mkdir /tmp/dir10-20
sftp> cd /tmp/dir10-20
sftp> pwd
Remote working directory: /tmp/dir10-20
sftp> put /etc/group
Uploading /etc/group to /tmp/dir10-20/group
group                                       100% 1118     1.0MB/s   00:00    
sftp> ls -l
-rw-r--r--    1 user1    user1        1118 Jul 21 01:41 group
sftp> cd ..
sftp> pwd
Remote working directory: /tmp
sftp> cd /home/user1
sftp> get /usr/bin/gzip
Fetching /usr/bin/gzip to gzip
gzip                                        100%   90KB  23.0MB/s   00:00    
sftp> 
```

- `lcd`, `lls`, `lpwd`, and `lmkdir` are run on the source server. 
- Other commands are also available. (See man pages)

Type quit at the sftp\> prompt to exit the program when you're done:
```bash
sftp> quit
[user1@server30 tmp]$ 
```

## Secure Shell Service DIY Labs

### Lab: Establish Key-Based Authentication 

- Create user account user20 on both systems and assign a password. 
```bash
[root@server40 ~]# adduser user20
[root@server40 ~]# passwd user20
Changing password for user user20.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
```

- As user20 on server40, generate a private/public key pair without a passphrase using the ssh-keygen command. 
```bash
[user20@server40 ~]# ssh-keygen -N "" -q
Enter file in which to save the key (/root/.ssh/id_rsa): 
```

- Distribute the public key to server30 with the ssh-copy-id command. 
`[user20@server40 ~]# ssh-copy-id server30`
- Log on to server30 as user20 and accept the fingerprints for the server if presented. 
```bash
[user20@server40 ~]# ssh server30
Activate the web console with: systemctl enable --now cockpit.socket

Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Fri Jul 19 14:09:22 2024
[user20@server30 ~]# 
```

- On subsequent log in attempts from server40 to server30, user20 should not be prompted for their password. 

### Lab: Test the Effect of PermitRootLogin Directive 

- As user1 with sudo on server30, edit the /etc/ssh/sshd_config file and change the value of the directive PermitRootLogin to "no". 
`[user1@server30 ~]$ sudo vim /etc/ssh/sshd_config` 

- Use the systemctl command to activate the change. 
```bash
[user1@server30 ~]$ systemctl restart sshd
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to restart 'sshd.service'.
Authenticating as: root
Password: 
==== AUTHENTICATION COMPLETE ====
```

- As root on server40, run ssh server40 (or use its IP). You'll get permission denied message. 

(this didn't work, I think it's because I configured passwordless authentication on here)

- Reverse the change on server40 and retry ssh server40. You should be able to log in. 

