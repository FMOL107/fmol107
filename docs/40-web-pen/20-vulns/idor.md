---
title: IDOR – Detection & Exploitation
description: Detección y explotación de Insecure Direct Object Reference (IDOR)
---

# IDOR – Detection & Exploitation

Una **Insecure Direct Object Reference (IDOR)** ocurre cuando una aplicación expone referencias directas a objetos internos (IDs, rutas, nombres de archivo, etc.) sin validar si el usuario tiene permisos para acceder a ellos.

Este fallo permite a un atacante acceder a **recursos de otros usuarios** simplemente manipulando identificadores.

---

## Concepto

Ejemplos comunes de objetos expuestos:
- IDs numéricos (`/data/1`, `/file/42`)
- UUIDs predecibles
- Nombres de archivo
- Rutas directas a recursos

El problema no es el identificador en sí, sino la **ausencia de control de acceso**.

---

## Indicadores de IDOR

Durante la enumeración web, prestar atención a:

- Parámetros numéricos incrementales
- URLs del tipo:
	- /object/&lt;id&gt;
	- /download?id=123  
	- /user/42
- Respuestas válidas al modificar el ID manualmente
- Recursos accesibles sin autenticación o con autenticación mínima

---
## Metodología de detección

1. Interactuar normalmente con la aplicación
2. Identificar recursos generados dinámicamente
3. Observar el patrón del identificador
4. Modificar el valor manualmente (±1, valores bajos, valores altos)
5. Comparar respuestas del servidor

---

## Ejemplo típico

Recurso generado por el usuario:

```text
/data/4
```

Pruebas manuales:

`/data/3 /data/2 /data/1 /data/0`

Si el servidor devuelve recursos válidos sin restricción, existe un IDOR.

---

## Explotación

Una vez confirmado el IDOR:
- Enumerar recursos existentes
- Descargar o visualizar información sensible
- Buscar credenciales, tokens, configuraciones o datos internos

Ejemplos:
- Capturas PCAP
- Facturas
- Documentos privados
- Logs
- Backups

---

## Impacto

El impacto depende del tipo de objeto accesible:
- **Bajo**: Información pública de otros usuarios
- **Medio**: Datos internos, configuraciones
- **Alto**: Credenciales, secretos, información sensible

En escenarios reales, los IDOR suelen conducir a:
- Compromiso de cuentas
- Movimiento lateral
- Acceso inicial al sistema

---

## Prevención (defensivo)
- Validar autorización en el backend
- No confiar en IDs enviados por el cliente
- Usar controles de acceso por objeto
- Evitar IDs predecibles

---

## Referencias

- [04-Testing_for_Insecure_Direct_Object_References](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
- [https://portswigger.net/web-security/access-control/idor](https://portswigger.net/web-security/access-control/idor)