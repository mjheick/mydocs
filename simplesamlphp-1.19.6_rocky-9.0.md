# Installing SimpleSAMLphp 1.19.6 on Rocky Linux 9.0
This is my quick go-to guide on installing [SimpleSAMLphp 1.19.6](https://github.com/simplesamlphp/simplesamlphp/releases/download/v1.19.6/simplesamlphp-1.19.6.tar.gz) on [Rocky Linux 9.0](https://download.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9.0-20220805.0-x86_64-minimal.iso)

# Install Rocky Linux 9.0
Perform the standard installation, doing updates.
- root password is _password_
- log in as root
- yum update
- yum install epel-release wget
- reboot

# Install Apache
- sudo yum install httpd php mod_ssl
- 
