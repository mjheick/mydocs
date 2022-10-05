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

## Metadata Certificates
Create and move metadata certificates into place:
```
openssl req -x509 -newkey rsa:1024 -keyout server.pem -out server.crt -sha256 -nodes -days 3650
sudo cp server.* /var/www/html/simplesamlphp-1.19.6/cert
sudo chown apache:apache /var/www/html/simplesamlphp-1.19.6/cert/server.*
```

This is mainly needed for the IDP to work but can also be done on the SP for it's own certificates.

## Front-End Test
IdP: [https://192.168.1.221/module.php/core/frontpage_welcome.php]()

SP: [https://192.168.1.163/module.php/core/frontpage_welcome.php]()

# SAML Time

## Metadata Exchange IdP->SP
IdP Metadata: [https://192.168.1.221/saml2/idp/metadata.php?output=xhtml]()

Place this data in saml20-idp-remote.php file on the SP

```sudo -u apache vi /var/www/html/simplesamlphp-1.19.6/metadata/saml20-idp-remote.php```

## Metadata exchange SP->IDP
SP Metadata: [https://192.168.1.163/module.php/saml/sp/metadata.php/default-sp?output=xhtml]()

Place this data in the saml20-sp-remote.php file on the IdP

```sudo -u apache vi /var/www/html/simplesamlphp-1.19.6/metadata/saml20-sp-remote.php```

## Setup the correct login auth on the IDP

bBy default the login module set up on IdPs is ```example-userpass```.

Edit saml20-idp-hosted.php and change ```'auth'```'s value from ```example-userpass``` to ```admin```. We already have _admin_ credentials for this with the password configured as _password_.

```sudo -u apache vi /var/www/html/simplesamlphp-1.19.6/metadata/saml20-idp-hosted.php```

## Login to 221 from 163
Now we should be able to do a SAML login from the SP at 163 to the IDP at 221 with _admin_/_password_:
- [https://192.168.1.163/module.php/core/authenticate.php]()
- Select _default-sp_
- Select the IDP at _https://192.168.1.221/saml2/idp/metadata.php_
- use the password _password_, select Login
- Return to 163 and be shown the attributes provided from 221
