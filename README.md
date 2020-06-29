#Disaster Recovery using duplicity and .gpg Encryption
In so many applications and projects, customers expect from the providers to offer a secure and robust back up strategy for the databases and even source codes.
There are so mnay different types of backup strategy with pros and cons. Selecting a proper stratefy mainly depends on the application, customer demand and potentioal dire situations.

In the following link, more than 5 different backup methods are illustrated.
Here, encrypted backup using duplicity is elaborated in practice for those who find the available explanatory blogs and posts relatively ambigious or out of date.

Link: https://www.digitalocean.com/community/tutorials/how-to-choose-an-effective-backup-strategy-for-your-vps

## Problem Statement
As developers, we have an application or data-base with 100% up-time. Therefore, the DB data is subjected to ongoing updates and refreshes. The customer demands to have secure disaster recovery plan which protect their data from lose due to reasons such as :

* operating system crashes and bugs
* Force-major incidents
* Hostile attacks
* potential data purge during development, etc.

In this tutorial, we do not assume any particular cloud host such as Digital ocean or AWS, and we try to be as general as possible. However, we assume that all the codes and DBs are deployed on the Ubuntu systems.

## Steps

### Pre-requisites
we assume that we want to back up one folder or some files inside that folder to a remote destination located in another ubuntu server. At this point there are two possibilities,
* Remote server is located in the same subnet that the host server exists and the conection can be established through private IP
* Remote server is outside of the subnet and the connection is through public IP

We focus on the later one as the genral case.


server_1 = Host server that we want to mounth the back up on


server_2 = The remote server that will host the backed up files


1. We need to establish a proper password less authuntication in the server_2 (remote server). To this aim we have to log in to the server_2 as root and set up a password if you have not done tha yet using
```
passwd
```
This comand will  let you to set up the root password for server_2 which will be required to set up the password less ssh connection from server_1 to server_2

2. Next is to log in to the server_1 (source server) and set up the ssh key using;
```
ssh-keygen -t rsa
```
After answering yes to the questions and creating the ssh key, we need to transfer it to the server_@ for password less acess. To this aim type the following command;
```
ssh-copy-id root@server_2_IP
```
this will ask for the passsword you set up on the server 2 in the previous step.

Now the ssh should be set up and it wont ask for the password next time you log in. Remember we need this for automated back up tha twe will explain later.
you can test the ssh connection using the following from the server_1 to server_2. 
```
ssh root@serever_2
```

no type ```exit``` and go back to server_1

2. In the next step, we need to install an stable version of the duplicity on server_1. currently the latest stable version is https://launchpad.net/duplicity/0.8-series/0.8.13/+download/duplicity-0.8.13.tar.gz, although there exist proceeding version which may be in ther alpha phase.
dowload the latest version https://launchpad.net/duplicity.
```
apt update && apt upgrade
apt install -y python3-boto python3-pip haveged gettext librsync-dev
wget https://code.launchpad.net/duplicity/0.8-series/0.8.12/+download/duplicity-0.8.12.1612.tar.gz
tar xaf duplicity-0.8.*.tar.gz
cd duplicity-0.8.*/
```
Then we install all the requirement listed in the downloaded folder using
```
pip3 install -r requirements.txt
```
and finally install the duplicity,
```
python3 setup.py install
```

3. next step is to crete the GPG keys. there are three important keys you need to save and remember here. public key, private key and the passphrase.The commands will store our keys in a hidden directory at /root/.gnupg/:
```
gpg --gen-key
```
You will be asked a series of questions that will configure the parameters of the key pair. Enter the name, email address, and optionally, a comment that will be associated with this key. Type O to confirm your information.
Then it will ask for the passphrases and after entring those, an entroy analysis will be done to generate the key-pairs using the system randomeness. The the following results after sometime will be shown;
```
gpg: /home/sammy/.gnupg/trustdb.gpg: trustdb created
gpg: key <your-GPG-public-key-id> marked as ultimately trusted
public and secret key created and signed.
```
in order to save the key pair and the pass-phrase in a folder and do not type it all the time we are getting a backup, we run,
```
mkdir ~/.duplicity
```
to make a direction. then,
```
nano ~/.duplicity/.env_variables.conf
```
to create a file to define the variables, which we will use by ```export``` stetment. This statement makes the variable avaialable to entire program.


The folloing is what you need to write in this file.

```
export GPG_KEY="your-GPG-public-key-id"
export PASSPHRASE="your-GPG-key-passphrase"
```
Then save it and set the permission
```
chmod 0600 ~/.duplicity/.env_variables.conf
```
and 
```
source ~/.duplicity/.env_variables.conf
```

### Run a file backup/restore scenario
In this part we would like to run a scenario of backing up a .sql file and assume we lost it for some reason and restore it fron the remote destination. If you do not have a ```.sql``` at hand to use, you can create a simple .txt file using ``` touch test.txt``` instead.


1. Create a 'backup_source' directory in your root folder in server_1
2. put the .sql or any other target files in that directory
3. Create another directory in the 'backup_source' and name it 'test_folder'
4. add some random files in that folder (e.g. some text or .sql file)


Run the folloing command to back up the .sql file you put into 'backup_source' directory
```
duplicity --encrypt-sign-key=$GPG_KEY \
/root/backup_source/a.sql \
sftp://root@server_2_IP//root/backup_destination/
```

This will create an encrypted back up from ```a.sql``` database only. If you log in to the server_2 and look into the backup_destination folder you will see,
```
-rw-r--r-- 1 root root 1143 Jun 29 18:48 duplicity-full-signatures.20200629T184821Z.sigtar.gpg
-rw-r--r-- 1 root root 1073 Jun 29 18:48 duplicity-full.20200629T184821Z.manifest.gpg
-rw-r--r-- 1 root root 1810 Jun 29 18:48 duplicity-full.20200629T184821Z.vol1.difftar.gpg
```

Now we would like to delete the a.sql file from the 'backup_source' and only restore that file. 

With this aim first delete the a.sql using ```rm -r a.sql``` in the server_1 bachup_source directory and then run the folloiwng command
```
duplicity --encrypt-sign-key=$GPG_KEY \
sftp://root@142.93.152.0//root/backup_destination/ /root/backup_source/a.sql
```

This command will restore the missing a.sql file back again.


In the next drill, we want to back up the entire backup_source folder and then try two differen things, 
1. Restore a specific file or folder
2. Restore the entire folder

With this aim we can run the following command which will back up the entire 'backup_source' folder to the server_2.


Note: before doing this, we need to delete the content of the 'backup_destination' folder.Otherwise, it will throw and error that requires force-full action.
```
duplicity --encrypt-sign-key=$GPG_KEY \
/root/backup_source \
sftp://root@server_2_IP//root/backup_destination/
```


now we fo to the bachup_source folder and empty the contents. Then we runf the folloing command.
```
duplicity --encrypt-sign-key=$GPG_KEY \
sftp://root@142.93.152.0//root/backup_destination/ /root/backup_source
```

** Note** As a rule of thubm the following is the format of this command for restoring file or directories
```
duplicity --encrypted-sign-key=$GPG_KEY [if empty, it will restore the entire backup folder]or[a relative destination to the particular file according to the bachup_source folder] \
sftp://root@142.93.152.0//[path to the folder of the backup_destination] [path to the folder that we wants the back up]
```


and if we want to restore only the a.sql file in the enire backedup folder, we run the following
```
duplicity --encrypt-sign-key=$GPG_KEY --file-to-restore a.sql \
sftp://root@142.93.152.0//root/backup_destination/ /root/backup_source
```


























## How to copy a file from another server
```
scp [switch] [source content location] [destination content location]
```

```
scp -r user1@www.server2.com:/var/www/ /var/www/dir
```

