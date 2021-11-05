### Virtual Hosting

*  /etc/apache2/sites-available/misitio.com.conf

```apacheconf
<VirtualHost *:80>
    ServerAdmin webmaster@misitio.com
    DocumentRoot "/var/www/html/misitio.com"
    ServerName misitio.com
    ServerAlias www.misitio.com
</VirtualHost>
```

* a2ensite mismito.com

* /etc/hosts

  ```bash
  172.17.0.2	misitio.com
  ```

### Autenticación 

#### Básica

* htpasswd -c etc/apache2/.htpasswd carlos

* /etc/apache2/sites-available/000-default.conf

  ```
  <Directory "/var/www/html/privado">
          AuthType Basic
          AuthName "Acceso Restringido"
          AuthUserFile /etc/apache2/.htpasswd
          Require valid-user
  </Directory>
  ```

* ```bash
  cd /var/www/html
  mkdir privado
  ```

* service apache2 reload

#### Básica con ficheros htaccess

* /etc/apache2/sites-available/000-default.conf

  ```apacheconf
  <Directory /var/www/html>
      Options Indexes FollowSymLinks
      AllowOverride All
      Require all granted
  </Directory>
  ```

* ```bash
  cd /var/www/html
  mkdir privado
  echo hola > index.html
  ```

* /var/www/html/privado/.htaccess

  ```
  AuthType Basic
  AuthName "Acceso Restringido"
  AuthUserFile /etc/apache2/.htpasswd
  Require valid-user 
  ```

#### Digest

a2enmod auth_digest

* /etc/apache2/sites-available/000-default.conf

  ```apacheconf
  <Directory /var/www/html/privado>
      AuthUserFile /etc/apache2/claves/digest.txt
      AuthName "dominio"
      AuthType Digest
      Require valid-user
  </Directory>
  ```

* ```bash
  htdigest [-c] /etc/apache2/claves/digest.txt dominio carlos2
  ```

### Mapeo de URL

* Opciones: Indexes, FollowSymLinks AllowOverride

* Alias
```apacheconf
  Alias /apache /etc/apache2
  <Directory /etc/apache2>
    Options Indexes
    Require all granted
  </Directory>
  ```

* Redirección con Redirect
```
  Redirect /mipagina http://localhost/info.php
```
* Redirección con Rewrite
```
  RewriteEngine on
  RewriteRule "^/bienvenido\.html$" "/listado.php" [R]
```
* Reescritura de URLs con Rewrite
```
  RewriteEngine on
  RewriteBase /
  RewriteRule "^bienvenido\.html$" "listado.php" [PT]
```

* Personalización de páginas de error
```
  ErrorDocument 404 /var/www/errores/error404.html
```

* Seguridad. Configurar HTTPS

Crear directorio, clave privada y certificado:

```
mkdir /etc/apache2/certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/certs/apache2.key -out /etc/apache2/certs/apache2.crt

---
Country Name (2 letter code) [AU]: ES 
State or Province Name (full name) [Some-State]: Madrid
Locality Name (eg, city) []: Madrid
Organization Name (eg, company) [Internet Widgits Pty Ltd]: IES Quevedo
Organizational Unit Name (eg, section) []: 1daw
Common Name (e.g. server FQDN or YOUR name) []: 1daw.quevedo.es
Email Address []: carlosgs@1daw.quevedo.es
---
```
Añadir al fichero del virtual host default-ssl.conf las directivas necesarias
```
<VirtualHost *:443>
    SLEngine On
    SSLCertificateFile /etc/apache2/certs/apache2.crt
    SSLCertificateKeyFile /etc/apache2/certs/apache2.key
    SSLProtocol All -SSLv3
</VirtualHost>
```
Y al 000-default.conf una redirección para que todo el tráfico del 80 vaya al 443
```
<VirtualHost *:80>
   Redirect / https://localhost/
</VirtualHost>
```
Cambiar el docker-compose.yml para que publique también el puerto 443

Habilitar el módulo ssl y el sitio, y recargar la configuración
```
a2enmod ssl
a2ensite default-ssl
service apache2 reload
```

* Configurar como servidor webDAV
```
a2enmod dav dav_fs
mkdir /var/www/webdav
chown -R www-data:www-data /var/www/webdav

DavLockDB /tmp/DavLock
Alias /webdav /var/www/webdav
<Directory /var/www/webdav>
  DAV On
  Options Indexes FollowSymLinks MultiViews
  AllowOverride None
  AuthType digest
  AuthUserFile "/etc/apache2/claves/digest.txt"
  AuthName "dominio"
  Require valid-user
</Directory>
```

* Configurar Proxy Inverso 
Usar el docker-compose.apache1-2.yml
a2enmod proxy proxy_http proxy_html

```
ProxyPass "/agencia/" "http://agencia.com/"
ProxyPassReverse "/agencia/" "http://agencia.com/"

ProxyHTMLURLMap http://agencia.com /agencia
<Location /agencia/>
    ProxyPassReverse /
    ProxyHTMLEnable On
    ProxyHTMLURLMap / /agencia/
</Location>
```

En apache2 añadir un fichero index.html con este contenido y comprobar si hace el cambio de url
```
<html>
<head></head>
<body>
<a href="http://agencia.com/mipagina.html">hola desde apache2</a>
</body>
</html>
```