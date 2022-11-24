## Proyecto Apache Virtual Host

Objetivos

El objetivo de esta practica es crear un dns para la pagina maravillosas.fabulas.com que redirija al cliente a una virtual host creada por nosotros

Configuración

Para configurar dicho dns vamos a ulitlizar nuestro contenedor apache que creamos en la pactica "Proyecto Apache" y vamos a darle un resolutor de dns reutilizando el codigo de la practica "Dns Linux".

Primero comenzamos añadien a nuestro docker-compose.yml un contenedor con un cliente alpine, el cual usaremos para conectarnos a la web, y un contenedor con una imagen bind9, el cual usaremos para resolver los nombres
```
  asir_cliente:
      container_name: asir_cliente2
      image: alpine
      networks:
        bind9_subnet:
          ipv4_address: 10.1.0.224
      stdin_open: true #docker run -i
      tty: true        #docker run -t
      dns:
          - 10.1.0.222

  bind9:
    container_name: asir_bind
    image: internetsystemsconsortium/bind9:9.16
    networks:
      bind9_subnet:
        ipv4_address: 10.1.0.222
    volumes:
       - ~/Escritorio/proyectoApache-master/config:/etc/bind
       - ~/Escritorio/proyectoApache-master/zonas:/var/lib/bind
networks:
    bind9_subnet:
        external: true
 ```
Una vez hecho esto debemos asegurarnos de craer la network bind9 con el comando **__docker network create --subnet=10.1.0.222/24 --gateway 10.1.0.1**

Tambien añadiremos dos carpetas una config, para confirar las zonas y una zonas que gestonara la resolución de nombres

- Config
```
zone "fabulas.com" {
        type master;
        file "/var/lib/bind/db.fabulas.com";
        allow-query{
            any;
            };
};
```
```
options {
        directory "/var/cache/bind";
        forwarders {
                8.8.8.8;
                8.8.4.8;
        };
        forward only;

        listen-on { any; };
        listen-on-v6 { any; };
        allow-query {
                any;
        };
        allow-recursion {
                none;
        };
        allow-transfer {
                none;
        };
        allow-update {
                none;
        };
};
```  
- Zonas
```

$TTL    3600
@       IN      SOA     ns.fabulas.com. root.fabulas.com. (
                   2007010401           ; Serial
                         3600           ; Refresh [1h]
                          600           ; Retry   [10m]
                        86400           ; Expire  [1d]
                          600 )         ; Negative Cache TTL [1h]
;
@       IN      NS      ns.fabulas.com.
ns       IN      A       10.1.0.222

maravillosas     IN      A       10.1.0.225
oscuras     IN      CNAME       maravillosas
```
A continuacion nos dirigimos a nuestra carpeta Sites enable y añadimos nuestro server name como a continuación. Para conectarlo con el puerto indicado en mi caso el 80:

![This is an image](https://github.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/blob/main/imagenes/Captura%20de%20pantalla%20de%202022-11-17%2019-59-57.png)


Si hemos hecho todo bien ya podremos hacer un docker-compose up y ya tendremos nuestro set-up completo

Comprobación
![This is an image](https://github.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/blob/main/imagenes/Captura%20desde%202022-11-17%2020-21-43.png)

Ahora para comprobarque funciona debemos entrar en la shell de nuestro cliente e intentaremos hacer ping a la ip nuestro apache y a la ip nuestro de servidor dns


![This is an image](https://github.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/blob/main/imagenes/Captura%20desde%202022-11-17%2020-01-05.png)

Una vez comprobamos que tenemos conexión con el dns ya podremos hacer peticiones a la página y recebiremos la información que introduimos dentro imagen

## Configurar SSL

Para empezar nos dirigimos a  attach shell del apache y escribimos el comando **__"a2enmod ssl"__**, esto activara la configuración de nustro certificados en nuestro contenedor.

A continuación creamos la carpeta **__"certs"__**, donde guardaremos nuestros certificados, y nos movemos a dicha carpeta  

Ahora para completar dicha configuración  utilizamos el comando **__"openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out apache-certificate.crt -keyout apache.key"__** y rellenamos los campos como se indique

Ahora debemos avtivar la configuración Modificar el fichero default-ssl.conf con:
a2ensite default-ssl.conf

Tambien creamos una carpeta SSL, con un index dentro que sera lo que aparecera cuando cargemos el host

- **Modificacionn del fichero default-ssl.conf**

```

    DocumentRoot /var/www/html/ssl --- Nueva ruta
```
    
![This is an image](https://github.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/blob/main/imagenes/Captura%20de%20pantalla%20de%202022-11-23%2019-52-11.png)

    SSLCertificateFile    /etc/apache2/certs/apache-certificate.crt  --- Nuevas rutas
    SSLCertificateKeyFile /etc/apache2/certs/apache.key
    
![This is an image](https://github.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/blob/main/imagenes/Captura%20de%20pantalla%20de%202022-11-23%2019-52-42.png)

-  *Recordar Editar docker para añadir puerto 443*
- Si hemos completado todos los pasos correctamente nuestro ssl ya deber
![This is an image](https://github.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/blob/main/imagenes/Captura%20de%20pantalla%20de%202022-11-23%2020-29-40.png)

## Añadir firefox

Para añadir firefox lo que vamos a hacer es añadir las siguientes lineas de codigo
```
  firefox:
    container_name: firefox
    image: jlesage/firefox
    ports:
      - '5800:5800'
    volumes:
      - ./Firefox:/config:rw
    dns:
      - 10.1.0.222
    networks:
      bind9_subnet:
        ipv4_address: 10.1.0.233
 ```
 Tambien creamos una nueva carpeta firefox donde se van a guardar los ficheros del volumen
 **__mkdir firefox__**
 
 Si hemos hecho todo correctamente nuestro contenedor firefox ya debería funcionar
 ![This is an image](https://github.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/blob/main/imagenes/Captura%20desde%202022-11-24%2016-38-58.png)

## Añadir Wireshark

Para añadir un contenedor de Wireshark añadimos el siguiente codigo a nuestro codigo a nuestro **__Docker Compose__**

```
  wireshark:
    image: lscr.io/linuxserver/wireshark:latest
    container_name: wire
    cap_add:
      - NET_ADMIN
    security_opt:
      - seccomp:unconfined 
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - ./wires:/config
    ports:
      - 3000:3000
    restart: unless-stopped
```

 Tambien creamos una nueva carpeta firefox donde se van a guardar los ficheros del volumen
 **__mkdir wires__**
 
 Si hemos hecho todo correctamente nuestro contenedor wireshark ya debería funcionar
 ![This is an image](https://github.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/blob/main/imagenes/Captura%20desde%202022-11-24%2017-59-15.png)
