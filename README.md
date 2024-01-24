# p6-jenkinsAWS
Practica 6 Jenkins - DAW - Despliegue´


# Instalar Jenkins en docker
```bash
* Crear un bridge network o puente de red con el comando:
docker network create jenkins
* Crear un dockerfiles con los datos:
*Construir la imagen con el sgte comando:
docker build -t myjenkins-blueocean:2.426.2-1 .
ejecutamos la imagen creada como un contenedor con el comando docker run : 

```
# Iniciamos Jenkins  
```bash
En el navegador localhost:8080 la cual nos pedirá una clave

```
# Recuperamos la contraseña
```bash
En la terminal entramos al contenedor
docker exec -it <id del contenedor> bash
buscamos la carpeta
cd /var/jenkins_home/secrets/
abrimos el archivo
cat initialAdminPassword
Le damos click a Install suggested plugins
Se instala y luego pedira registrarnos
Le damos en en skip, y confirmamos el puerto y listo

```
# Crear un nuevo proyecto
```bash
Una vez en el panel de Jenkins le damos click a create a job
Digitamos el nombre del proyecto
Buscamos la opción : pipeline
ok

```
# Traer el código. Deberemos tener un git con la configuración previamente hecha
```bash
stage('clean') {
            steps {
                sh 'rm -rf *'
            }
stage('clone') {
            steps {
                sh 'git clone https://github.com/mikkype/StoreAWS.git'
            }
        }
Clean : Para borrar el contenido de nuestro proyecto
Clone : para clonar el repositorio de github ,que contiene el src y el Dockerfile

```
# Build del docker con el código dentro
```bash
stage('build') {
            steps {
                sh 'cd StoreAWS && docker build -t migueperu/miguelweb:latest .'
            }
        }
Build : Debemos tener permisos root :
docker exec -it -u <idcontenedor> bash : 
chown root:jenkins /var/run/docker.sock
sudo groupadd docker
sudo usermod -aG docker ${USER}
reboot




para construir la imagen que va estar dentro de la carpeta del repositorio de github,agregamos el nombre de nuestra imagen que será el mismo de nuestro registry de docker-hub con un tagname en esta caso le llamamos “latest”.

```

# Push de la imagen de docker al registry de docker
```bash
stage('login') {
            steps {
                sh 'cat ~/mypassword.txt | docker login --username migueperu --password-stdin'
            }
        }
        stage('push') {
            steps {
                sh 'docker push migueperu/miguelweb:latest'
            }
        }


login : dentro del contenedor de jenkins debemos crear un archivo con nuestro token-credencial generado en  docker hub. 
cd /var/jenkins_home => nano mypassword.txt => insertar el token
push : Aquí hacemos el push de la imagen creada anteriormente hacia el registry de docker hub.

Una vez configurado el proyecto le damos en construir ahora
Y luego ya tendremos nuestra proyecto jenkins creado y subido al registry de docker hub :
En caso de errores revisar el console log.

```
# Desarrolla los archivos de configuración para hacer un despliegue en AWS
```bash
Entramos a AWS en servicios EC2 , lanzar instancias ,donde configuramos el despliegue en servidor:
Lanzar instancia: Creamos un nombre de nuestra instancia
AMI : escogemos Ubuntu - Capa gratuita
Tipo de estancia : t2.micro apto capa gratuita
Par de  claves : generamos par de claves ,le asignamos un nombre y de tipo .emp , generar y nos descarga el archivo lo cual debemos instalarlo dentro de nuestro sistema de raíz ubuntu : usuario/home
Lo cual podemos trabajar allí o desde el mismo servidor de aws.
Configuraciones de red : marcar las tres casillas 
Finalmente el damos click a Lanzar Instancia
Nos aparece un panel con la estancia creada,la seleccionamos y le damos a Conectar


Conectarse a la instancia: es aquí donde obtenemos el id del  cliente SSH ,entramos y seguimos los pasos , y copiamos el ssh para trabajarlo en nuestra terminal ubuntu, le damos a Conectar y se nos abrirá una web con el servidor de aws.

Dentro de la terminal : 
traemos la imagen creada de nuestro repositorio github desde el docker hub : 
docker pull <imagendockerhub>
Construimos la imagen descargada: 
docker build -t <imagendockerhub> .
Corremos el contenedor en el puerto 80 con la imagen construida : 
docker run -dit –name <nombrecontendor> -p 80:80 <imagendockerhub>
En el panel de instancias de aws buscamos nuestra IP
seleccionamos la instancia , luego en Acciones/Ver detalles

copiamos la dirección IPv4 pública y la pegamos en el navegador
y finalmente tenemos nuestro pagina web 


```

```bash

```