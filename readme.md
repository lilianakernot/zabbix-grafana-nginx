# Zabbix + Grafana + Nginx como Reverse Proxy

## Descripción
Este proyecto despliega un stack de monitoreo con **Zabbix Server, Web, Agent** y **Grafana**, detrás de un reverse proxy Nginx con HTTPS (certificado autofirmado con SAN).

## Servicios
- Postgres (DB para Zabbix)
- Zabbix Server
- Zabbix Web
- Zabbix Agent
- Grafana
- Nginx (reverse proxy SSL)

## Pasos principales
1. Editar archivo hosts en Windows
Abrir como administrador el archivo:
C:\Windows\System32\drivers\etc\hosts

Agregar al final:
127.0.0.1   zabbix.local
127.0.0.1   grafana.local

Guardar cambios y vaciar caché DNS:
ipconfig /flushdns

2. Generar certificados SSL autofirmados con SAN (`zabbix.local`, `grafana.local`).Comandos openssl:
openssl req -nodes -x509 -sha256 -newkey rsa:4096   -keyout grafana.key   -out grafana.crt   -days 3650   -subj "/C=país/ST=provincia/L=localidad/O=empresa/OU=IT/CN=grafana.local"   -addext "subjectAltName = DNS:grafana.local"

openssl req -nodes -x509 -sha256 -newkey rsa:4096   -keyout zabbix.key   -out zabbix.crt   -days 3650   -subj "/C=país/ST=provincia/L=localidad/O=empresa/OU=IT/CN=zabbiz.local"   -addext "subjectAltName = DNS:zabbix.local"
Copiar ambos archivos dentro de /zabbix-grafana-nginx/nginx/certificados-ssl/

3. Crear archivos de configuración de grafana.local y zabbix.local para Nginx que contienen:
El bloque configura del servidor HTTPS
Rutas a los certificados
location:
	proxy_pass= Redirige las peticiones al contenedor(se especifica el puerto)
	proxy_set_header Host $host; = Reenvía el encabezado original host al backend.
	proxy_set_header X-Real-IP $remote_addr; =Inserta la IP real del cliente en un encabezado HTTP
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; = Añade la IP real del cliente a la cadena X-Forwarded-For
	proxy_set_header X-Forwarded-Proto $scheme; = Indica si el cliente se conectó por http o https
Copiar ambos archivos dentro de zabbix-grafana-nginx/nginx/conf.d

4.Levantar el stack con docker-compose:
Base de datos (PostgreSQL)para zabbix.local
Zabbix Server
Zabbix Web
Zabbix Agent
Grafana
Nginx (reverse proxy)
con sus respectivos volúmenes con red docker llamada net-proxy.
driver: bridge que significa que es una red interna de docker tipo bridge
Copiar el archivo dentro de /zabbix-grafana-nginx/

Correrlo con: docker compose up -d

5.Validación
Acceder a Zabbix:
https://zabbix.local
Usuario: Admin / Password: zabbix

Acceder a Grafana:
https://grafana.local
Usuario: admin / Password: admin

El navegador mostrará advertencia por certificado autofirmado → aceptar excepción.
