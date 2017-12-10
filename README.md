
# HomeBox
A set of Ansible scripts to setup your personal mail server (and more) for your home...

## Folders:
- config: Ansible hosts file Configuration.
- preseed: Ansible scripts to create an automatic ISO image installer for the base system.
- install: Ansible scripts to install the mail server environment.

## Preseed folder
This step is only required if you do not have a ready to use Debian server, and you want to quickly setup one.

The preseed folder contains scripts to create an iso imag for Debian Stretch, with automatic installation.
It is set to install your system on two disks with software RAID and LVM,
although this setup will be made optional.
It can be used both for development with a VM or for production to install the operating system base.

### 1. Create the hosts.yml configuration
There is two hosts that need to be defined. One that will run the Ansible scripts,
and one that will be used as the mail server.

- Copy the file config/hosts-example.yml into config/host.yml
- Update your hosts definition

I personally use my workstation to run the Ansible scripts, and a virtual machine with snapshots for development.

### 1. Setup SSH authentication:
Copy your public key in `preseed/misc/root/.ssh/authorized_key`. This file is ignored by git.
This key will be copied into the `/root/.ssh/authorized_keys` by the automatic installer
for you to connect to your Linux server

### 2. Customisation:
Copy common.example.yml to common.yml, and modify the values accordingly.

- The disk names will need to be modified for an automatic installation
  - For a virtual machine, the disks might be called vda and vdb, but sda and sdb for a physical server.
  - The network configuration, especially the domain name you want to use
  - The country and the locale values
  - The timezone
  - The root password

### 3. Build the ISO image
Then, inside the preseed folder, run this command to build the ISO image:

`ansible-playbook -v -i ../config/hosts.yml playbooks/build-cd.yml`

This will create the ISO images in /tmp folder. Use the DVD one for automatic installation.

### 4. Use a physical server or a VM to run the Debian installer.
The whole installation should be automatic, with LVM and software RAID
For LVM, there is a volume called "reserved" you can remove. This will let
you resize the other volumes according to your needs.


## Mail server installation

First, check that you can login onto your mail server, using SSH on the root account:

`ssh root@mail.example.com`

Replace mail.example.com with the address of your mail server.
This will also add the server to your known_hosts file

### Copy the example files to create your basic setup

Run the following code to create your custom files

```
cd install/playbook/variables
cp accounts.example.yml accounts.yml
cp common.example.yml common.yml
cp dovecot.example.yml dovecot.yml
cp postfix.example.yml postfix.yml
cp webmail.example.yml webmail.yml
```

File contents
- common.yml: common variables and basic network configuration (domain)
- accounts.yml: User and group accounts information
- dovecot.yml: Dovecot mail server configuration
- postfix.yml: Postfix MTA configuration
- webmail.yml: Webmail (roundcube) configuration

### Run the Ansible scripts to setup your email server
The installation folder is using Ansible to setup the mail server
For instance, inside the install folder, run the following command:

`ansible-playbook -vv -i ../config/hosts.yml playbooks/install.yml`

The script is actually doing following:

- Install some required packages.
- Install a simple LDAP server (openLDAP).
- Create a valid certificate, and activate TLS authentication for the LDAP server.
- Create the user and group accounts in the directory.
- Integrate the LDAP accounts into the system using pam_ldap and nslcd (optional).
- Install Postfix mail transfer agent, with a dedicted SSL certificate.
- Create a 2048 bits DKIM key, and publish the associated DNS record.
- Update the SPF records with your external IP address.
- Install Dovecot mail server, with a dedicted SSL certificate for IMAP.
- Install PostgreSQL for the database.
- Install Roundcube with nginx, and create a dedicated SSL certificate for the webmail.

The certificates are generated using LetsEncrypt service, with one for each service. Examples for example.com domain:
  - openldap: ldap.example.com
  - postfix: smtp.example.com
  - dovecot: imap.example.com
  - the webmail: webmail.example.com

## What do you need for production usage?
### Basic requirements

- A low consumption hardware box to plug on your router.
- A computer to run the Ansible scripts.
- A static IP address from your ISP (Not mandatory, but works better)
- Some basic newtork knowledge

For automatic DNS records creation and update:
- A registered domain name, with [Gandi](https://gandi.net/)

### Automatic update of the DNS entries
A custom script automatically register domains entries to Gandi, if you provide an API key.
This is useful so you do not have to manage the entries yourself, or if even if you don't have a static IP address.

Here what to do to obtain an API key:

1. Visit the [API key page on Gandi](https://www.gandi.net/admin/api_key)
2. Activate of the API on the test platform.
3. Activate of the production API

Your API key will be activated on the test platform.

#### Notes:
- The initial creation of DNS records for certificate generation should take some time, I am working on a solution.
- DNS automatic update is actually limited to Gandi, but it should be easy to add more.

