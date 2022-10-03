# Installing SimpleSAMLphp 1.19.6 on Rocky Linux 9.0
This is my quick go-to guide on installing [SimpleSAMLphp 1.19.6](https://github.com/simplesamlphp/simplesamlphp/releases/download/v1.19.6/simplesamlphp-1.19.6.tar.gz) on [Rocky Linux 9.0](https://download.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9.0-20220805.0-x86_64-minimal.iso)

# Install Rocky Linux 9.0
Perform the standard installation, doing updates.
root password is _password_

log in as root

```yum update```

```yum install epel-release wget tar```

reboot

turn off firewalld and set selinux to permissive in /etc/selinux/config

# Download SimpleSAMLphp
```wget https://github.com/simplesamlphp/simplesamlphp/releases/download/v1.19.6/simplesamlphp-1.19.6.tar.gz```

```tar -xf simplesamlphp-1.19.6.tar.gz```

# Install and Configure Apache
```sudo yum install httpd php mod_ssl```

```sudo mv ~/simplesamlphp-1.19.6 /var/www/html```

```chown apache:apache /var/www/html/simplesamlphp-1.19.6```

```chown -R apache:apache /var/www/html/simplesamlphp-1.19.6```

create /etc/httpd/conf.d/simplesamlphp.conf with contents
```
<VirtualHost 192.168.1.x:80>
    ServerName 192.168.1.x
    DocumentRoot /var/www/html/simplesamlphp-1.19.6/www
</VirtualHost>

<VirtualHost 192.168.1.x:443>
    ServerName 192.168.1.x
    DocumentRoot /var/www/html/simplesamlphp-1.19.6/www
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
</VirtualHost>
```

```sudo systemctl start httpd```

# Configure SimpleSAMLphp
edit ```/var/www/html/simplesamlphp-1.19.6/config/config.php```

```
'baseurlpath' => '/',
'secretsalt' => 'secretsalt',
'auth.adminpassword' => 'password',
'technicalcontact_email' => 'user@example.org',
```
on the IdP set: ```'enable.saml20-idp' => true,```

Create and move IdP metadata certificates into place:
```
openssl req -x509 -newkey rsa:1024 -keyout server.pem -out server.crt -sha256 -nodes -days 3650
sudo cp server.* /var/www/html/simplesamlphp-1.19.6/cert
sudo chown apache:apache /var/www/html/simplesamlphp-1.19.6/cert/server.*
```

# Metadata Exchange
IdP Metadata: [https://192.168.1.x/saml2/idp/metadata.php?output=xhtml]

SP Metadata: [https://192.168.1.x/module.php/saml/sp/metadata.php/default-sp?output=xhtml]
