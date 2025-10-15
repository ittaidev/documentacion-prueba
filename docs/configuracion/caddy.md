# Caddy

## Introducción  
Caddy es un servidor web y proxy inverso de código abierto, conocido por su implicidad, ligereza y facilidad de configuración.  
En el proyecto bitCLD, se utiliza exclusivamente para la gestión del tráfico interno dentro de la red local (LAN), actuando como punto de acceso unificado a los diferentes servicios autohospedados mediante dominios internos del tipo `*.lan`.

---

## Motivos de elección  
La elección de Caddy respondió a la necesidad de contar con una herramienta **ligera, segura y fácil de mantener**, en línea con los pilares de bitCLD: **simplicidad, seguridad y autohospedaje**.  

Caddy permite gestionar varios servicios de forma ordenada mediante un único archivo de configuración (`Caddyfile`), sin depender de configuraciones complejas como las de Nginx o Apache.  
Además, su naturaleza **open source** y su diseño minimalista lo hacen ideal para equipos con recursos limitados.

Ventajas clave

- Configuración limpia y declarativa  
- TLS interno automático  
- Integración directa con Docker  
- Ligero en recursos  
- Comunidad activa y documentación clara

---

## Función dentro del proyecto  
En bitCLD, Caddy actúa como **reverse proxy interno** dentro de la red local.  
Gestiona las conexiones entre los distintos servicios (Nextcloud, AdGuard, Dockge, Vaultwarden, Uptime-Kuma, etc.), ofreciendo acceso mediante dominios internos (`*.lan`) y **certificados TLS locales** generados automáticamente.

El tráfico externo **no pasa por Caddy**, sino que es gestionado directamente por [Cloudflare Tunnel](./cloudflare.md/), que se conecta a los puertos de los servicios.  
De esta forma, Caddy gestiona el entorno LAN mientras que Cloudflare controla el acceso remoto seguro.

---

## Despliegue con Docker  
Caddy se ejecuta en un contenedor Docker independiente, con su propia red interna `caddy_net`.  
Esto garantiza aislamiento, simplicidad y compatibilidad con el resto de servicios.

    services:
    caddy:
        image: caddy:latest
        container_name: caddy
        restart: unless-stopped
        ports:
        - 80:80
        - 443:443
        volumes:
        - ./Caddyfile:/etc/caddy/Caddyfile:ro
        - ./data:/data
        - ./config:/config
        networks:
        - caddy_net

    networks:
    caddy_net:
        external: true

### Explicación del docker compose

        image: caddy:latest
        container_name: caddy
        restart: unless-stopped
| Parámetro | Descripción | 
|----------|---------|
| `image: caddy:latest` | Usa la última versión oficial de la imagen de Caddy desde Docker Hub. | 
| `container_name: caddy` | Asigna un nombre fijo al contenedor (útil para comandos y depuración). | 
| `restart: unless-stopped` | Hace que el contenedor se reinicie automáticamente si se detiene de forma inesperada, salvo que se haya detenido manualmente. | 

        ports:
        - 80:80
        - 443:443
| Puertos expuestos | Descripción | 
|----------|---------|
| `80:80` | Expone el puerto 80 para tráfico HTTP. | 
| `443:443` | Expone el puerto 443 para tráfico HTTPS. | 

<em>`Estos mapeos permiten que el servidor Caddy atienda solicitudes web desde el exterior del contenedor.`</em>

        volumes:
        - ./Caddyfile:/etc/caddy/Caddyfile:ro
        - ./data:/data
        - ./config:/config
         
| Volumen | Descripción | 
|----------|---------|
| `./Caddyfile:/etc/caddy/Caddyfile:ro` | Archivo principal de configuración de Caddy. Se monta como solo lectura (ro) para mayor seguridad. | 
| `./data:/data` | Almacena certificados SSL y datos generados por Caddy. | 
| `./config:/config` | Guarda configuraciones internas de Caddy (por ejemplo, caché o estados). |

        networks:
        caddy_net:
            external: true

| Red | Descripción | 
|----------|---------|
|`caddy_net` | El servicio se conecta a la red caddy_net, lo que permite que Caddy se comunique con otros contenedores |
|`external: true` | Indica que la red caddy_net ya existe y no será creada por este docker-compose. Esto permite compartir la misma red entre distintos docker-compose.yml, haciendo posible que varios contenedores se comuniquen entre sí. |


---


## Configuración del Caddyfile

El archivo Caddyfile contiene los distintos subdominios internos y sus destinos locales.
Cada dominio usa **tls internal** para generar certificados automáticos válidos dentro de la red.

    # Nextcloud
    cld.lan {
        tls internal
        reverse_proxy 192.168.1.5:8080
    }

    # AdGuard Home
    dns.lan {
        tls internal
        reverse_proxy 192.168.1.5:3000
    }

    # Dockge
    dockge.lan {
        tls internal
        reverse_proxy 192.168.1.5:5001
    }

    # Uptime Kuma
    monitor.lan {
        tls internal
        reverse_proxy 192.168.1.5:3001
    }

    # Vaultwarden
    pass.lan {
        reverse_proxy vaultwarden:80
        tls internal
    }

    # Netdata
    stats.lan {
        tls internal
        reverse_proxy 192.168.1.5:19999
    }

    # IA Local y API
    chat.lan {
        tls internal
        reverse_proxy 192.168.1.5:3003
    }

---

## Integración con AdGuard Home (DNS interno)  
**AdGuard Home** cumple la función de **servidor DNS interno** dentro de bitCLD.  
Lo hemos configurado para que los dispositivos de la red puedan resolver los dominios locales (`*.lan`) hacia la IP del servidor principal ( `192.168.1.5`).  

Para ello, se definimos **rewrites personalizados**, de forma que cuando un dispositivo de la red consulta, por ejemplo, `dockge.lan`, el DNS le redirige correctamente al servicio correspondiente gestionado por Caddy.


<p align="center">
  <img src="../../assets/img/caddy/dns-rewrites.png" alt="DNS Rewrites de AdGuard Home" width="600">
</p>

<p align="center"><em>Rewrites de Adguard Home</em></p>

De este modo, cualquier dispositivo conectado a la red y usando AdGuard como DNS podrá acceder fácilmente a los servicios internos a través de sus dominios `.lan`.

### Para más información sobre el funcionamiento del DNS accede a la [documentación de Adguard Home](../servicios/adguard.md/)

---

## Integración sin necesidad de cambiar DNS

En Windows y Linux podemos evitar la necesidad de usar Adguard Home como DNS modificando un archivo, sin embargo, en móvil si que es obligatorio usar <em>bitCLD</em> como servidor DNS para que podamos usar los dominios `.lan`.

### Windows

En Windows se debe modificar el archivo hosts.

Para ello abrimos el bloc de notas como administrador --> Abrir... --> `C:\Windows\System32\drivers\etc\hosts` y seleccionamos "Todos los archivos" para que podamos verlo.

<p align="center">
  <img src="../../assets/img/caddy/windows-hosts.png" alt="C:\Windows\System32\drivers\etc\hosts" width="600">
</p>

<p align="center"><em>C:\Windows\System32\drivers\etc\hosts</em></p>

Dentro del archivo copiamos las rutas correspondientes

    192.168.1.5 dockge.lan
    192.168.1.5 cld.lan
    192.168.1.5 dns.lan
    192.168.1.5 monitor.lan
    192.168.1.5 stats.lan
    192.168.1.5 pass.lan
    192.168.1.5 chat.lan

Después de guardar los cambios en el archivo hosts, debemos vaciar la caché del DNS para que los cambios surtan efecto. Para ello abrimos el símbolo del sistema como administrador y ejecutamos el comando `ipconfig /flushdns`. Con este comando eliminamos las entradas temporales de la caché del DNS, forzando al sistema a realizar una nueva consulta de resolución de nombres desde el archivo hosts que acabamos de configurar.

---


## Certificados y acceso HTTPS interno

Para que los dispositivos de la red puedan acceder correctamente a los dominios locales con HTTPS, es necesario instalar el certificado raíz de Caddy en cada dispositivo que los utilice.
<p align="center">
  <img src="../../assets/img/caddy/no-segura.png" alt="Ejemplo de no tener instalado el certificado en el dispositivo" width="600">
</p>

<p align="center"><em>Ejemplo de no tener instalado el certificado en el dispositivo</em></p>


### Ubicación del certificado
El certificado raíz se encuentra en el contenedor de Caddy en:
`/data/caddy/pki/authorities/local/root.crt`
Para copiarlo en el host abrí una terminal y usé el comando 
`scp user@192.168.1.5:/srv/docker/stacks/caddy/data/caddy/pki/authorities/local/root.crt`
                                                    FALTA IMAGEN DE ESO

### Proceso de instalación

#### En Windows


Debemos ir a certmgr.msc --> Entidades de certificación raíz de confianza --> Todas las tareas --> Importar

<p align="center">
  <img src="../../assets/img/caddy/w+r.png" alt="Ejecutar en Tecla Windows + R" width="400">
</p>

<p align="center"><em>Ejecutar en Tecla Windows + R</em></p>

<p align="center">
  <img src="../../assets/img/caddy/importar.png" alt="Importar certificado en Windows" width="600">
</p>



Una vez descargado veremos que ya reconoce el certificado y que se establece una conexión segura.

<p align="center">
  <img src="../../assets/img/caddy/certificado-instalado.png" alt="Importar certificado en Windows" width="600">
</p>

<p align="center"><em>Certificado instalado</em></p>


  
## En Android

En Android, a diferencia de Windows o Linux, es obligatorio usar Adguard Home como servidor por lo que primero debemos asegurarnos que está como nuestro servidor DNS.

<p align="center">
  <img src="../../assets/img/caddy/android-ip.jpg" alt="Establecer Adguard Home como servidor DNS" width="300">
</p>

<p align="center"><em>Establecer Adguard Home como servidor DNS</em></p>

Una vez estamos usando nuestro servidor como servidor DNS puedemos observar que tenemos acceso a dockge.lan, pero que nuestro dispositivo no reconoce el certificado de caddy y que nos muestra como conexión no segura.

<p align="center">
  <img src="../../assets/img/caddy/android-conexin-nossl.jpg" alt="Ejemplo de no tener instalado el certificado en el dispositivo" width="600">
</p>

<p align="center"><em>Ejemplo de no tener instalado el certificado en el dispositivo NO TENGO LA IMAGEN</em></p>

Para instalar certificado debemos irnos a Privacidad > Más ajustes > Encriptación y credenciales > Instalar certificado





- En Linux: Añade el .crt en /usr/local/share/ca-certificates/ y ejecuta update-ca-certificates.

Sin este paso, los navegadores marcarán los dominios .lan como “no seguros”, aunque la conexión sea realmente cifrada.

## Conclusión

Aunque su función es interna, Caddy es un componente esencial dentro de bitCLD.
Junto con [Adguard Home](../servicios/adguard.md/), ofrece una experiencia fluida para los dispositivos de la red, permitiendo el acceso seguro y organizado a todos los servicios mediante dominios locales y certificados válidos.

Su combinación con [Cloudflare Tunnel](./cloudflare.md/) para el acceso remoto crea una infraestructura equilibrada y coherente, donde cada herramienta cumple una función clara dentro de los pilares del proyecto: simplicidad, seguridad y autohospedaje.