<p align="center">
  <img src="{{ 'assets/img/general/logonobg.png' | relative_url }}" alt="Logo bitCLD" width="400">
</p>

# Plataforma Autogestionada de Servicios en la Nube
## Descripción General

Este proyecto implementa una infraestructura completa de servicios autogestionados en un entorno cloud, 
combinando contenedores Docker, servicios de productividad, monitorización, y seguridad perimetral. 
Se incluye una VPN privada para acceso remoto seguro, gestión de correo profesional, y un reverse proxy 
para exposición controlada de servicios. Además, integra capacidades de inteligencia artificial mediante 
APIs y modelos locales, garantizando privacidad y flexibilidad. 

La solución elimina la necesidad de abrir 
puertos hacia internet, centralizando la administración y ofreciendo alta disponibilidad con un enfoque 
en la ciberseguridad y eficiencia operativa.
		

## Servicios Incluidos

| Servicio | Función | Puerto | Red Local | Subdominio | 
|----------|---------|--------|------------|---------|
| Nextcloud | Almacenamiento cloud | 8080 | cld.lan | cld.bitcld.com | 
| Dockge | Gestor de Docker | 5001 | dockge.lan | dockge.bitcld.com | 
| AdGuard Home | DNS + Ad-blocking | 53, 3000 | dns.lan | | 
| Uptime Kuma | Monitorización | 3001 | monitor.lan | monitor.bitcld.com | 
| Netdata | Métricas del sistema | 19999 | stats.lan | stats.bitcld.com | 
| Vaultwarden | Gestor de contraseñas | 3002 | pass.lan | pass.bitcld.com | 
| Inteligencia artificial | Local Y API | 3003 | chat.lan | chat.bitcld.com | 

## Diagrama de la infraestructura

<p align="center">
  <img src="../../assets/img/general/diagrama-completo.png" alt="Diagrama de la infraestructura bitcld" width="600">
</p>

<p align="center"><em>Diagrama de bitCLD</em></p>












