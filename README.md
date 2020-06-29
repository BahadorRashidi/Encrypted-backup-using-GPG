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


