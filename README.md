# React con Docker en Oracle Cloud

Proyecto: `D:\Proyecto Sistemas III\Oracle-Cloud-and-React\react-app`

Instancia Ubuntu: `152.67.34.169` · Usuario SSH: `ubuntu`

## Resultado verificado — 10 de octubre de 2026

- React 19.2.8 y Vite 8.3.4 instalados.
- Revisión de código y compilación satisfactorias en Windows y dentro de Docker.
- Contenedor `oracle-react-web` ejecutándose y con estado `healthy`.
- Respuesta HTTP 200 de `http://152.67.34.169` comprobada desde Windows.
- Página renderizada y contador de 0 a 1 comprobados en el navegador mediante un túnel SSH temporal al mismo contenedor. El navegador integrado agotó el tiempo de espera al abrir directamente la IP pública.
- Docker activado al arrancar Ubuntu y contenedor configurado con reinicio automático.
- Permisos de la clave SSH original corregidos para que solo tu usuario de Windows tenga acceso.

Captura de la prueba: `react-oracle-verificado.png`, junto a esta guía en la carpeta de entregables.

Se instaló la plantilla oficial de React con Vite y JavaScript. La página inicial incluye un contador para comprobar que React funciona. El Dockerfile compila con Node y sirve el resultado con Nginx. No hace falta instalar Node en Ubuntu: Docker lo usa durante la construcción.

## Ejecutar React en Windows para desarrollar

En PowerShell:

```powershell
Set-Location 'D:\Proyecto Sistemas III\Oracle-Cloud-and-React\react-app'
npm.cmd install
npm.cmd run dev
```

Abre la dirección que indique Vite, normalmente `http://localhost:5173`. Detén el servidor con `Ctrl+C`. Modifica `src/App.jsx` para cambiar la página.

## Revisar el código y compilar

```powershell
npm.cmd run lint
npm.cmd run build
```

El resultado se guarda en `dist`. El archivo `package-lock.json` conserva las versiones instaladas; el Dockerfile usa `npm ci` para reproducirlas.

## Ejecutar Docker en Windows

Primero inicia Docker Desktop con contenedores Linux. Luego, desde la carpeta `react-app`:

```powershell
docker compose up -d --build
docker compose ps
```

Abre `http://localhost:8080`. Para detener este proyecto:

```powershell
docker compose down
```

## Conectarse a la máquina virtual

En PowerShell:

```powershell
ssh -i 'D:\Proyecto Sistemas III\Oracle-Cloud-and-React\ssh-key-2026-10-11.key' ubuntu@152.67.34.169
```

La clave privada debe tener acceso únicamente para tu usuario de Windows. No compartas su contenido ni la subas a Docker Hub o a un repositorio.

## Consultar React en Ubuntu

Los siguientes comandos se ejecutan después de entrar por SSH:

```bash
cd /home/ubuntu/oracle-cloud-react
sudo docker ps
sudo docker logs --tail 30 oracle-react-web
curl -f http://localhost/health
curl -I http://localhost
```

Imagen: `oracle-react:1.0`. Contenedor: `oracle-react-web`. Puerto: `80:80`. Política de reinicio: `unless-stopped`.

Para reconstruir tras actualizar los archivos, ejecuta primero la construcción; solo si termina bien, reemplaza el contenedor:

```bash
sudo docker build -t oracle-react:1.0 .
```

```bash
sudo docker stop oracle-react-web
sudo docker rm oracle-react-web
sudo docker run -d --restart unless-stopped -p 80:80 --name oracle-react-web oracle-react:1.0
```

## Flujo de la clase con Docker Hub (opcional)

En este despliegue se transfirió el código por SSH y se construyó la imagen en Ubuntu. Si el profesor exige subir la imagen a Docker Hub, usa tu propia cuenta; el usuario del compañero no es tu cuenta.

En PowerShell, con Docker Desktop iniciado y desde `react-app`:

```powershell
$usuarioDockerHub = Read-Host 'Tu usuario de Docker Hub'
docker login
docker build -t "${usuarioDockerHub}/oracle-react:1.0" .
docker push "${usuarioDockerHub}/oracle-react:1.0"
```

En Ubuntu, sustituye `TU_USUARIO` por el mismo usuario. Descarga primero la imagen y, si termina bien, reemplaza el contenedor:

```bash
sudo docker pull TU_USUARIO/oracle-react:1.0
```

```bash
sudo docker stop oracle-react-web
sudo docker rm oracle-react-web
sudo docker run -d --restart unless-stopped -p 80:80 --name oracle-react-web TU_USUARIO/oracle-react:1.0
```

## Errores corregidos del texto de la clase

- `-p 8080:80` conecta el puerto 8080 de Windows con el 80 del contenedor. `-p 8080` no expresa ese mapeo.
- `--name` recibe el nombre del contenedor; la imagen se escribe al final de `docker run`.
- `docker stop` y `docker rm` reciben nombres o identificadores de contenedores.
- Un contenedor detenido sigue reservando su nombre hasta eliminarlo.
- Se escribe `latest`, no `lastest`, si se usa esa etiqueta.
- La URL correcta es `http://localhost`.
- Copia solo los comandos, sin `PS ...>`, `ubuntu@...$` ni los resultados de consola.
- React debe compilarse antes de servirlo con Nginx. Copiar directamente JSX a Nginx no ejecuta React.

## Acceso desde Internet

Dirección prevista: `http://152.67.34.169`. Para acceder públicamente, la lista de seguridad o el NSG de Oracle debe permitir ingreso TCP al puerto de destino 80. Docker publica el puerto del contenedor en la máquina virtual.

## Documentación oficial

- [React: crear una aplicación desde cero](https://react.dev/learn/build-a-react-app-from-scratch)
- [Vite: instalación y plantillas](https://vite.dev/guide/)
- [Docker: construcción en varias etapas](https://docs.docker.com/get-started/docker-concepts/building-images/multi-stage-builds/)
- [Oracle: listas de seguridad](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securitylists.htm)
