# Zoho Mail

## Introducción  
**Zoho Mail** es una plataforma de correo electrónico profesional que forma parte del ecosistema de aplicaciones en la nube de Zoho.  
A diferencia de otros servicios de correo tradicionales, Zoho ofrece un entorno sin anuncios, con una interfaz moderna y una buena integración con dominios personalizados.  

En el proyecto **bitCLD**, Zoho Mail se utiliza para gestionar las cuentas de correo con dominio propio (`@bitcld.com`), permitiendo un contacto profesional y un entorno de pruebas funcional para el envío y recepción de correos desde el dominio del proyecto.

Zoho Mail **no es un servicio autohospedado**, sino que funciona sobre la infraestructura de **Zoho Corporation**, utilizando sus servidores en la nube. Esto permite alta disponibilidad, simplificar la configuración y disponer de un servicio estable sin necesidad de mantener infraestructura propia de correo.

---

## Motivos de elección  
La elección de **Zoho Mail** se basó principalmente en su **plan gratuito para dominios personalizados**, que permite crear hasta **5 cuentas de correo sin coste**, con almacenamiento suficiente y funcionalidades avanzadas.  

**Ventajas principales**

- Permite usar un dominio propio sin costes adicionales  
- Sin anuncios y con interfaz limpia  
- Compatibilidad con IMAP/POP3 y aplicación web moderna  
- Panel de administración sencillo y completo  
- Ideal para proyectos personales o en fase de desarrollo  

Esta opción se consideró ideal para **bitCLD**, al ofrecer una alternativa profesional y funcional sin necesidad de mantener un servidor SMTP propio ni abrir puertos hacia internet.

---

## Función dentro del proyecto  
Dentro del entorno **bitCLD**, **Zoho Mail** cumple una función doble:

1. **Correo de contacto principal**, vinculado al dominio del proyecto (`contacto@bitcld.com`), para comunicaciones formales y documentación.  
2. **Plataforma de pruebas**, para validar configuraciones de DNS, SPF, DKIM y DMARC con un dominio real y medir la correcta autenticación de los correos.  

Aunque bitCLD prioriza el autohospedaje, se decidió integrar un proveedor externo de correo por motivos de **seguridad, reputación de dominio y mantenimiento**, evitando la necesidad de desplegar un servidor de correo propio.

---

## Instalación y configuración  

A continuación se resumen los pasos seguidos para la configuración del dominio `bitcld.com` con Zoho Mail:

### 1. Registro en Zoho Mail  
- Acceder a [https://www.zoho.com/es-xl/mail/](https://www.zoho.com/es-xl/mail/)
- Seleccionar la opción de Business Email  
- Seleccionar el plan **"Mail Lite – Free for custom domains"** (gratuito hasta 5 usuarios)  
- Crear una cuenta de administrador

### 2. Verificación del dominio  
Zoho solicita comprobar que somos propietarios del dominio.  
Esto se realiza añadiendo un **registro TXT** en la configuración DNS (en nuestro caso, está gestionado por **Cloudflare**):

        Tipo: TXT

        Nombre: <el que ofrece zoho>

        Valor: <código_de_verificación>

Una vez añadido, ya podremos verificarlo desde el panel de Zoho.

### 3. Configuración de registros MX  
Para recibir correos correctamente, es necesario añadir los registros **MX** que Zoho indica:

| Tipo | Nombre | Prioridad |
|------|---------|------------|
| MX | @ | 10 | mx.zoho.eu |
| MX | @ | 20 | mx2.zoho.eu |
| MX | @ | 50 | mx3.zoho.eu |


### 4. Autenticación (SPF, DKIM)  
Para mejorar la entrega y seguridad del correo, se configuraron los siguientes registros:

#### SPF
v=spf1 include:zoho.eu ~all

#### DKIM
Zoho genera un registro tipo TXT con nombre `zoho._domainkey` y un valor único, que debe añadirse al DNS.


Una vez hayamos finalizado, los registros deben aparecer así:

<p align="center">
  <img src="../../assets/img/zoho/registros-dns.png" alt="Registros DNS completos" width="900">
</p>

<p align="center"><em>Registros DNS completos</em></p>


**En caso de usar Cloudflare como gestor de dominio. Para que funcione correctamente debemos desactivar el _proxy nube naranja_ en estos registros.**

Los servicios de correo requieren resolución directa hacia los servidores de Zoho.

<p align="center">
  <img src="./assets/img/zoho/nube-proxy.png" alt="No usar cloudflare como proxy" width="300">
</p>

## Prueba de funcionamiento  

Una vez completada la configuración de los registros DNS y la verificación del dominio, tendremos una cuenta de correo funcional.

<p align="center">
  <img src="./assets/img/zoho/cuenta-zoho.png" alt="Cuenta Zoho" width="400">
</p>

Ahora vamos a realizar pruebas básicas de funcionamiento para comprobar la correcta entrega y recepción de correos bajo el dominio `@bitcld.com`.

### 1. Envío de correo de prueba  
Se envió un mensaje desde una de las cuentas configuradas (`contacto@bitcld.com`) hacia una cuenta externa, en mi caso `proton mail` pero podría ser cualquier otro (Gmail, Outlook...).

<p align="center">
  <img src="./assets/img/zoho/prueba-envio1.png" alt="Cuenta Zoho" width="700">
</p>

<p align="center">
  <img src="./assets/img/zoho/prueba-envio2.png" alt="Cuenta Zoho" width="700">
</p>

### 2. Recepción de correo  
Realizamos una prueba inversa, enviando un correo desde `bitcld@proton.me` hacia la dirección `contact@bitcld.com`, comprobando que:

- El mensaje llega al buzón en Zoho Mail sin retrasos ni errores.  
- La interfaz web de Zoho muestra correctamente los mensajes y notificaciones.  

<p align="center">
  <img src="./assets/img/zoho/prueba-recepcion1.png" alt="Cuenta Zoho" width="700">
</p>

<p align="center">
  <img src="./assets/img/zoho/prueba-recepcion2.png" alt="Cuenta Zoho" width="700">
</p>

### Acceso vía aplicación móvil  
Zoho Mail ofrece acceso tanto desde su aplicación web ([mail.zoho.eu](https://mail.zoho.eu)) como desde sus apps móviles oficiales.  
Verificamos la sincronización en tiempo real desde la aplicación.

<p align="center">
  <span>
    <img src="./assets/img/zoho/movil1.jpg" alt="Cuenta Zoho móvil" width="300" style="margin-right: 10px;">
  </span>
  <span>
    <img src="./assets/img/zoho/movil2.jpg" alt="Bandeja de entrada móvil" width="300">
  </span>
</p>



---

## Conclusión  
La integración de **Zoho Mail** en **bitCLD** aporta un componente esencial de comunicación profesional sin necesidad de autohospedar un servidor de correo, lo que simplifica la gestión y mejora la fiabilidad.  

Gracias a su plan gratuito para dominios personalizados, ofrece una solución ideal para proyectos de pequeño y mediano tamaño que buscan **identidad propia, estabilidad y bajo mantenimiento**.  

Aunque se apoya en una infraestructura externa, y por tanto no es autoalojado, su implementación complementa la filosofía del proyecto: **mantener un ecosistema funcional, seguro y eficiente**, priorizando el control y la simplicidad sobre la complejidad técnica innecesaria.

Además gracias a Zoho, se consigue una **suite completa y funcional para el trabajo colaborativo**, complementaria a [Nextcloud](../servicios/nextcloud.md/) que combina productividad, comunicación y autogestión bajo un mismo entorno unificado.

En resumen, Zoho Mail se integra en bitCLD como un servicio externo confiable, que aporta valor añadido al proyecto sin comprometer la autonomía ni la seguridad del entorno autogestionado.


---
