# Installing SimpleSAMLphp 1.19.6 on Rocky Linux 9.0
This is my quick go-to guide on installing [SimpleSAMLphp 1.19.6](https://github.com/simplesamlphp/simplesamlphp/releases/download/v1.19.6/simplesamlphp-1.19.6.tar.gz) on [Rocky Linux 9.0](https://download.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9.0-20220805.0-x86_64-minimal.iso)

# Rocky Linux 9.0
This will be our operating system of choice for this, so we'll manually set it up from the ISOs

## Standard Installation:
- kdump disabled
- automatic partitioning
- root password is _password_
- idp with static IP, ending octet 221 (I+D+P)
- sp with static IP, ending octet 163 (S+P)

## Basic Configuration
Reboot the systems, log in as root, do updates and set up users:
- ```yum -y update```
- ```reboot```
- ```sed -i s/=enforcing/=permissive/g' /etc/selinux/config```
- ```systemctl stop firewalld && systemctl disable firewalld```
- ```yum -y install epel-release wget tar```
- ```adduser user && passwd user```
- set password to _password_, cause why not
- run _visudo_, add ```user ALL=(ALL) ALL``` at bottom.

And a final reboot.

# Install and Configure Apache
```sudo yum -y install httpd php mod_ssl```

```sudo touch /etc/httpd/conf.d/simplesamlphp.conf```

## IdP Apache Configuration
Place the following into ```/etc/httpd/conf.d/simplesamlphp.conf```
```
<VirtualHost 192.168.1.221:80>
    ServerName 192.168.1.221
    DocumentRoot /var/www/html/simplesamlphp-1.19.6/www
</VirtualHost>

<VirtualHost 192.168.1.221:443>
    ServerName 192.168.1.221
    DocumentRoot /var/www/html/simplesamlphp-1.19.6/www
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
</VirtualHost>
```

## SP Apache Configuration
Place the following into ```/etc/httpd/conf.d/simplesamlphp.conf```
```
<VirtualHost 192.168.1.163:80>
    ServerName 192.168.1.163
    DocumentRoot /var/www/html/simplesamlphp-1.19.6/www
</VirtualHost>

<VirtualHost 192.168.1.163:443>
    ServerName 192.168.1.163
    DocumentRoot /var/www/html/simplesamlphp-1.19.6/www
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
</VirtualHost>
```

## Start the process

```sudo systemctl start httpd```

# SimpleSAMLphp

## Download and Install
- ```wget https://github.com/simplesamlphp/simplesamlphp/releases/download/v1.19.6/simplesamlphp-1.19.6.tar.gz```
- ```tar -xf simplesamlphp-1.19.6.tar.gz```
- ```sudo mv ~/simplesamlphp-1.19.6 /var/www/html```
- ```sudo chown apache:apache /var/www/html/simplesamlphp-1.19.6```
- ```sudo chown -R apache:apache /var/www/html/simplesamlphp-1.19.6```

## Configure SimpleSAMLphp
edit ```/var/www/html/simplesamlphp-1.19.6/config/config.php``` and place the following at the bottom

```sudo -u apache vi /var/www/html/simplesamlphp-1.19.6/config/config.php```

```
$config['baseurlpath'] = '/';
$config['secretsalt'] = 'secretsalt';
$config['auth.adminpassword'] = 'password';
$config['technicalcontact_email'] = 'user@example.org';
```
on the IdP add: ```$config['enable.saml20-idp'] = true;```

## IdP Certificates
Create and move IdP metadata certificates into place:
```
openssl req -x509 -newkey rsa:1024 -keyout server.pem -out server.crt -sha256 -nodes -days 3650
sudo cp server.* /var/www/html/simplesamlphp-1.19.6/cert
sudo chown apache:apache /var/www/html/simplesamlphp-1.19.6/cert/server.*
```

# Front-End Test
IdP: [https://192.168.1.221/module.php/core/frontpage_welcome.php]()

SP: [https://192.168.1.163/module.php/core/frontpage_welcome.php]()

# Metadata Exchange
IdP Metadata: [https://192.168.1.221/saml2/idp/metadata.php?output=xhtml]

SP Metadata: [https://192.168.1.163/module.php/saml/sp/metadata.php/default-sp?output=xhtml]
