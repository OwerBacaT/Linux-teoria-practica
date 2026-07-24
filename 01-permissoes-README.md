# 01 - Permisos y Control de Acceso en Linux

## Objetivo
Simular un escenario realista de auditoría de permisos de archivos en un servidor Linux, identificando y corrigiendo configuraciones inseguras, uno de los fallos más comunes encontrados en pentests y auditorías de hardening.

## Entorno
- WSL con CentOS
- Estructura simulando una carpeta de aplicación con archivo de credenciales, script y reporte

## Escenario creado

```bash
mkdir -p ~/proyecto_seguridad/app
cd ~/proyecto_seguridad/app

echo "DB_USER=admin
DB_PASS=ContraseñaSuperSecreta123" > config_db.env

echo '#!/bin/bash
echo "Backup en ejecución..."' > backup.sh

touch reporte_publico.txt

chmod 777 config_db.env
chmod 644 backup.sh
chmod 600 reporte_publico.txt
```

## Estado inicial (auditoría)

| Archivo | Permiso | Problema identificado |
|---|---|---|
| `config_db.env` | `777` (rwxrwxrwx) | 🔴 Crítico — cualquier usuario del sistema podía leer y escribir las credenciales de la base de datos |
| `backup.sh` | `644` (rw-r--r--) | 🟡 Faltaba el bit de ejecución (`x`) para el dueño |
| `reporte_publico.txt` | `600` (rw-------) | 🟡 Debería ser público para lectura, pero solo el dueño tenía acceso |

## Correcciones aplicadas

```bash
chmod 600 config_db.env         # solo el dueño lee/escribe — credenciales protegidas
chmod 740 backup.sh             # dueño ejecuta, grupo lee/ejecuta, otros nada
chmod 644 reporte_publico.txt   # dueño edita, todos leen
```

## Estado final

```
-rw------- config_db.env
-rwxr----- backup.sh
-rw-r--r-- reporte_publico.txt
```

## Comandos de auditoría aprendidos

```bash
# Archivos que cualquier usuario puede leer
find ~/proyecto_seguridad -type f -perm -o+r

# Archivos que otros pueden escribir (bandera roja)
find ~/proyecto_seguridad -type f -perm -o+w

# Archivos ejecutables por cualquier usuario
find ~/proyecto_seguridad -type f -perm -o+x

# Archivos con permiso total (777)
find ~/proyecto_seguridad -type f -perm 777
```

## Principales aprendizajes

- Notación simbólica (`rwx`) vs numérica (`chmod 750`, por ejemplo) y cómo convertir una en la otra
- Diferencia entre `-perm -MODO` (al menos esos permisos), `-perm /MODO` (cualquiera de ellos) y `-perm MODO` (exactamente esos)
- `;` ejecuta comandos en secuencia sin importar el resultado; `&&` solo ejecuta el siguiente si el anterior tuvo éxito, importante en scripts de seguridad para no enmascarar fallos
- Bits de ejecución innecesarios (`x`) en archivos que no son scripts "contaminan" los escaneos de auditoría (ej: CIS Benchmark) y pueden ocultar archivos maliciosos reales

## Próximos pasos
- Usuarios y grupos (`useradd`, `groupadd`) para control de acceso basado en grupos
