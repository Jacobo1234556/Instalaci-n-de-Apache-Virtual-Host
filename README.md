# Proyecto Apache Virtual Host

**Objetivos**

El objetivo de esta practica es crear un dns para la pagina maravillosas.fabulas.com que redirija al cliente a una virtual host creada por nosotros

**Configuración**

Para configurar dicho dns vamos a ulitlizar nuestro contenedor apache que creamos en la pactica "_Proyecto Apache_" y vamos a darle un resolutor de dns reutilizando el codigo de la practica "_Dns Linux_".

Primero comenzamos añadien a nuestro **docker-compose.yml** un contenedor con un cliente alpine, el cual usaremos para conectarnos a la web, y un contenedor con una imagen bind9, el cual usaremos para resolver los nombres

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
Tambien añadiremos dos carpetas una **config**, para confirar las zonas y una **zonas** que gestonara la resolución de nombres
* Config

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
* Zonas
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
A continuacion nos dirigimos a nuestra carpeta _Sites enable_ y añadimos nuestro server name como a continuación. Para conectarlo con el puerto indicado en mi caso el 80:

![This is an image](https://raw.githubusercontent.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/ae18b4eee7cabef0fb31f71e45f56f2c9011740b/imagenes/Captura%20de%20pantalla%20de%202022-11-17%2019-59-57.png)

Si hemos hecho todo bien ya podremos hacer un *docker-compose up* y ya tendremos nuestro set-up completo

**Comprobación**

Ahora para comprobarque funciona debemos entrar en la shell de nuestro cliente e intentaremos hacer ping a la ip nuestro apache y a la ip nuestro de servidor dns

![imagen](https://user-images.githubusercontent.com/113456615/202537958-8062b198-ec7e-4064-ba54-566c287deec6.png)

Una vez comprobamos que tenemos conexión con el dns ya podremos hacer peticiones a la página y recebiremos la información que introduimos dentro
![imagen](https://github.com/Jacobo1234556/Instalacion_de_Apache_Virtual-Host/blob/main/imagenes/Captura%20desde%202022-11-17%2020-21-43.png?raw=true)
