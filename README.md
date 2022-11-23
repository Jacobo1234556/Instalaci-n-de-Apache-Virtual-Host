# Configurar SSL


Ir a attach shell del apache y escribir a2enmod ssl
Crear una carpeta llamada certs
moverse a certs 

Escribir esto y rellenar como se indique
openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out apache-certificate.crt -keyout apache.key

Modificar el fichero default-ssl.conf con:
a2ensite default-ssl.conf

Crear carpeta SSL y crear index dentro

DocumentRoot /var/www/html/ssl --- Nueva ruta

SSLCertificateFile	/etc/apache2/certs/apache-certificate.crt  --- Nuevas rutas
SSLCertificateKeyFile /etc/apache2/certs/apache.key

Editar docker para añadir puerto 443