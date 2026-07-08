# 🚀 Instalación de GitLab CE y GitLab Runner con Docker

> 📚 **Objetivo del laboratorio**
>
> Implementar un servidor **GitLab Community Edition (CE)** utilizando Docker y configurar **GitLab Runner** para ejecutar futuros pipelines de **Integración Continua (CI)** y **Entrega Continua (CD)**.

---

# 📑 Contenido

- 📦 Instalación de GitLab CE con Docker
- ⚙️ Configuración inicial de GitLab CE
- 🏃 Instalación de GitLab Runner en Zorin OS 18

---

# 📦 1. Instalación de GitLab CE con Docker

En esta sección se desplegará un servidor GitLab CE utilizando un contenedor Docker. Además, se configurarán directorios persistentes para conservar la información aunque el contenedor sea eliminado.

---

## 📁 Paso 1. Crear los directorios persistentes

GitLab almacena configuraciones, registros y repositorios. Para evitar perder esta información, se crearán directorios permanentes en el sistema anfitrión.

Ejecute los siguientes comandos:

```bash
sudo mkdir -p /srv/gitlab/config
sudo mkdir -p /srv/gitlab/logs
sudo mkdir -p /srv/gitlab/data
```

### 🔍 Verificar los directorios creados

```bash
ls -l /srv/gitlab
```

Resultado esperado:

```text
config
data
logs
```

> 💡 **Explicación**
>
> Estos directorios permanecerán en el sistema anfitrión y conservarán la información de GitLab incluso si el contenedor es eliminado o actualizado.

---

## 🐳 Paso 2. Desplegar GitLab CE

Crear el contenedor ejecutando:

```bash
docker run -d \
  --name gitlab-ce \
  --hostname gitlab \
  --network host \
  --restart always \
  --shm-size 256m \
  -v /srv/gitlab/config:/etc/gitlab \
  -v /srv/gitlab/logs:/var/log/gitlab \
  -v /srv/gitlab/data:/var/opt/gitlab \
  -e GITLAB_OMNIBUS_CONFIG="external_url 'http://192.168.100.72'" \
  gitlab/gitlab-ce:latest
```

> 💡 **Explicación**
>
> Este comando descarga (si es necesario) e inicia GitLab CE dentro de un contenedor Docker utilizando almacenamiento persistente.

> ⚠️ **Importante**
>
> Reemplace **192.168.100.72** por la dirección IP de su equipo anfitrión.

---

### 🌐 ¿Por qué se utiliza `--network host`?

El parámetro:

```text
--network host
```

permite que GitLab utilice directamente la red del sistema operativo anfitrión.

Esto significa que el acceso se realizará mediante:

```text
http://IP_DEL_HOST
```

Por ejemplo:

```text
http://192.168.100.72
```

---

## ✅ Paso 3. Verificar el contenedor

Comprobar que GitLab se encuentra ejecutándose.

```bash
docker ps
```

Debe aparecer un contenedor similar a:

```text
gitlab-ce
Up
```

> 💡 **Explicación**
>
> Este comando muestra todos los contenedores Docker que actualmente se encuentran en ejecución.

---

## 📜 Paso 4. Visualizar el proceso de inicialización

Durante el primer arranque GitLab configura automáticamente todos sus componentes internos.

Para observar este proceso utilice:

```bash
docker logs -f gitlab-ce
```

> ⏳ **Importante**
>
> La primera inicialización puede tardar entre **5 y 15 minutos** dependiendo de los recursos disponibles.

---

## ❤️ Paso 5. Verificar los servicios internos

Ingresar al contenedor y comprobar que todos los servicios se encuentren activos.

```bash
docker exec -it gitlab-ce gitlab-ctl status
```

Deberían observarse servicios similares a:

```text
gitaly
gitlab-workhorse
nginx
postgresql
puma
redis
sidekiq
```

> 💡 **Explicación**
>
> GitLab está compuesto por múltiples servicios internos. Todos ellos deben encontrarse en estado **run** para que la plataforma funcione correctamente.

---

### ⚠️ Si algún servicio no inicia

Ejecutar:

```bash
docker exec -it gitlab-ce gitlab-ctl reconfigure
```

Posteriormente:

```bash
docker exec -it gitlab-ce gitlab-ctl restart
```

Finalmente volver a comprobar:

```bash
docker exec -it gitlab-ce gitlab-ctl status
```

---

# ⚙️ 2. Configuración inicial de GitLab CE

Una vez que GitLab ha iniciado correctamente, es momento de acceder por primera vez a la plataforma.

---

## 🔑 Paso 1. Obtener la contraseña inicial

GitLab genera automáticamente una contraseña para el usuario **root**.

Ejecute:

```bash
docker exec -it gitlab-ce cat /etc/gitlab/initial_root_password
```

Obtendrá una salida similar a:

```text
Password: xxxxxxxxxxxxxxxxxxxxxxxxx
```

> 💡 **Explicación**
>
> Esta contraseña será utilizada únicamente durante el primer acceso a GitLab.

---

## 🌍 Paso 2. Acceder a la interfaz web

Abrir el navegador e ingresar a:

```text
http://192.168.100.72
```

> ⚠️ **Importante**
>
> Sustituya **192.168.100.72** por la dirección IP de su equipo anfitrión.

Utilice las siguientes credenciales:

**Usuario**

```text
root
```

**Contraseña**

La obtenida en el paso anterior.

> 💡 **Explicación**
>
> Después de autenticarse podrá comenzar a crear usuarios, grupos y proyectos.

---

## 🔒 Paso 3. Cambiar la contraseña del usuario root (Opcional)

Por motivos de seguridad se recomienda reemplazar la contraseña generada automáticamente.

Ejecute:

```bash
docker exec -it gitlab-ce gitlab-rake "gitlab:password:reset[root]"
```

Ingrese la nueva contraseña cuando el sistema la solicite.

> 💡 **Explicación**
>
> Esta operación reemplaza la contraseña inicial del usuario administrador (**root**) por una nueva definida por el usuario.

---

# 🏃 3. Instalación de GitLab Runner en Zorin OS 18

GitLab Runner será el encargado de ejecutar automáticamente los trabajos definidos en los pipelines CI/CD.

---

## 📥 Paso 1. Descargar el script oficial

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" -o script.deb.sh
```

> 💡 **Explicación**
>
> Este script configura el repositorio oficial desde donde se instalará GitLab Runner.

---

## ⚙️ Paso 2. Configurar el repositorio

En Zorin OS, GitLab puede detectar incorrectamente la distribución.

Para evitar este inconveniente se fuerza el uso del repositorio correspondiente a **Ubuntu Noble**.

```bash
sudo os=ubuntu dist=noble bash script.deb.sh
```

> 💡 **Explicación**
>
> Aunque Zorin OS está basado en Ubuntu, GitLab no siempre reconoce correctamente la distribución. Este comando evita dicho problema.

---

## 📦 Paso 3. Instalar GitLab Runner

Actualizar la información de paquetes:

```bash
sudo apt update
```

Instalar GitLab Runner:

```bash
sudo apt install gitlab-runner
```

> 💡 **Explicación**
>
> GitLab Runner será el componente encargado de ejecutar automáticamente los pipelines definidos en los proyectos GitLab.

---

## ✅ Paso 4. Verificar la instalación

Consultar la versión instalada:

```bash
gitlab-runner --version
```

Comprobar el estado del servicio:

```bash
sudo systemctl status gitlab-runner
```

Resultado esperado:

```text
Active: active (running)
```

> 💡 **Explicación**
>
> Si el servicio aparece en estado **active (running)** significa que GitLab Runner está listo para registrarse posteriormente con un proyecto GitLab.

---

# 🎯 Resultado esperado

Al finalizar esta guía se dispondrá de:

- ✅ GitLab CE ejecutándose mediante Docker.
- ✅ Interfaz web accesible desde un navegador.
- ✅ Usuario administrador disponible.
- ✅ GitLab Runner instalado y listo para ser registrado en futuros proyectos.
- ✅ Plataforma preparada para implementar pipelines de Integración Continua y Entrega Continua (CI/CD).

---
