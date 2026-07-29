#### Rhel

Config root: /etc/httpd
Primary config file:  /etc/httpd/conf/httpd.conf
Module config: conf.modules.d
virtual host config: conf.d
Log: /var/log/httpd
User: apache

httpd.conf
- used `include` to reference other files
- 3 categories
	- Global settings
	- virtualhost
	- instructions for requests that do not match a VirtualHost defenition

Virtual host examples:

![](../../images/Pasted%20image%2020240402045440.png)

![](../../images/Pasted%20image%2020240402045454.png)

Basic authentication in Apache is configured in Location or Directory blocks


SSL might have changed its name to TLS

![](../../images/Pasted%20image%2020240402045643.png)

Here, the TLS certificate and key are located in Linux’s central system location, /etc/ssl. The public certificates can be readable by anyone, but the key should be accessible only to the Apache master-process user, typically root. We prefer to set permissions to 444 for the certificate and 400 for the key.

All versions of the actual SSL protocol (precursor to TLS) are known to be insecure and should be disabled with the SSLProtocol directive, shown above.

few ciphers have known weaknesses. You can configure the web server’s supported ciphers with the SSLCipherSuite directive


Mozilla Server Side TLS guide is the best resource that we are aware of for staying current

on best practices for TLS. It also has a handy configuration syntax reference for Apache, NGINX, and HAProxy.

certbot:

/etc/letencrypt/

sectigo

private key
/etc/pki/tls/private/

/etc/httpd/conf/httpd.conf
Document root /var/www/html

/var/www/swim

IfModule mod_ssl.c

cat 00-ssl.conf 
`LoadModule ssl_module modules/mod_ssl.so`

cat /usr/lib64/httpd/modules/mod_ssl.so
(filled with gibberish)

/etc/httpd/conf.d/vhost_marstherm-le-ssl.conf point to the let's encrypt SSL config
```
[root@marstherm conf.d]# cat vhost_marstherm-le-ssl.conf 
<IfModule mod_ssl.c>
<VirtualHost *:443>
        ServerName marstherm.psi.edu
        Redirect / https://marstherm.psi.edu/
SSLCertificateFile /etc/letsencrypt/live/marstherm.psi.edu/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/marstherm.psi.edu/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
SSLCertificateChainFile /etc/letsencrypt/live/marstherm.psi.edu/chain.pem
</VirtualHost>
</IfModule>
```

/etc/httpd/conf.d/ssl.conf point to old ssl files


```
[root@marstherm conf.d]# cat vhost_swim-le-ssl.conf 
<IfModule mod_ssl.c>
<VirtualHost *:443>
        ServerName swim.psi.edu
        Redirect / https://swim.psi.edu/


  <Directory /var/www/swim/>
        AllowOverride FileInfo AuthConfig Limit Indexes
        Options Indexes FollowSymLinks
        IndexOptions FancyIndexing
  </Directory>


SSLCertificateFile /etc/letsencrypt/live/swim.psi.edu/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/swim.psi.edu/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
SSLCertificateChainFile /etc/letsencrypt/live/swim.psi.edu/chain.pem
</VirtualHost>
</IfModule>
```

added the ssl certs to the normal ssl fileand renamed to letsencryptkettest.ssl.conf

also moved the original ssl.conf to current.ssl.conf
