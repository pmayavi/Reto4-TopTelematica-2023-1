Crear el grupo de seguridad con las siguientes reglas:  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/SecurityGroup.png)
Crear la base de datos, standar create, MariaDB, Free tier, llenar las credenciales, el usuario de la nuestra es pmayav y su contraseña pmayav1234, y ponerle el grupo de seguridad que creamos  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/DataBase.png)
Tambien ponerle el nombre a la base de datos, este lo usaremos mas adelante  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/DataBaseNombre.png)
Crear el EFS, guardar su nombre de DNS para uso luego  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/EFS.png)
  
Crear una maquina de AWS la cual sera el template para las otras  
Ingresar los siguientes comandos:  
```sh
sudo apt-get remove docker docker-engine docker.io containerd runc
```
```sh
sudo apt update
```
```sh
curl -fsSL https://get.docker.com/ -o get-docker.sh
sudo sh get-docker.sh
```
```sh
sudo groupadd docker
```
```sh
sudo usermod -aG docker $USER
```
Reinciar la conexion co la maquina, y se toma la imagen de moodle de https://hub.docker.com/r/bitnami/moodle  
Llenar los campos con los correctos de la base de datos, MOODLE_USERNAME y PASSWORD pueden ser cualesquiera para iniciar sesion en el moddle, el MOODLE_DATABASE_HOST es el endpoint de la base de datos:  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/Endpoint.png)
MOODLE_DATABASE_NAME es el nombre que pusimos arriba, en este caso database_mariadb1, y se ingresan las credenciales y puerto
```sh
docker run -d --name moodle --restart unless-stopped -p 80:8080 -p 443:8443 \
 --env MOODLE_USERNAME=pmayav \
 --env MOODLE_PASSWORD=pmayav1234 \
 --env MOODLE_DATABASE_HOST=database-mariadb1.ciyw2xdadx7x.us-east-1.rds.amazonaws.com \
 --env MOODLE_DATABASE_NAME=database_mariadb1 \
 --env MOODLE_DATABASE_USER=pmayav \
 --env MOODLE_DATABASE_PASSWORD=pmayav1234 \
 --env MOODLE_DATABASE_PORT_NUMBER=3306 \
 --env BITNAMI_DEBUG=true \
 --volume /moodledata:/bitnami bitnami/moodle:latest
```
```sh
docker logs moodle -f -n -t --details
```
  
Para confirmar la conexion con la base de datos opcionalmente se puede:
```sh
sudo apt install mariadb-client-core-10.6 -y
echo "show databases" | mariadb -u pmayav -p -h database-mariadb.ciyw2xdadx7x.us-east-1.rds.amazonaws.com
```  
Para la conexion con el NFS:
```sh
cd /moodledata
sudo apt install nfs-common -y && sudo apt install cifs-utils -y
```
En este comando se ingresa el nombre de DNS que guardamos al crear el NFS
```sh
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-0032ec83d2f306181.efs.us-east-1.amazonaws.com:/ /moodledata
```
```sh
sudo nano /etc/fstab
```
En el archivo ingresar en la ultima linea: (Ingresar bien el nombre de DNS)
```sh
fs-0032ec83d2f306181.efs.us-east-1.amazonaws.com:/ /moodledata nfs nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport
```
Y acabar con los comandos:
```sh
mount -a
mount | grep nfs
```
Ahora en AWS se detiene la maquina y se crea su imagen  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/Image.png)
Se crea un template nuevo desde esta imagen en Instances -> Launch Templates -> Create Launch Template  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/Template.png)
Crear el Auto Scaling Group:  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/AutoScaling.png)
Seleccionar las Availability Zones and subnets, nosotros seleccionamos 3, la us-east-1d, e y f  
Vamos a crear el Load Balancer en esta misma pagina  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/LoadBalancer.png)
El Load balancer scheme es Internet-facing y en Listeners and routing se crea un target group  
Por ultimo se selecciona el tamaño del grupo  
![Diagrama de arquitectura](https://github.com/pmayavi/Reto4-TopTelematica-2023-1/blob/main/Imagenes/GroupSize.png)
Ya finalizamos, al crearse correctamente el Auto Scaling Group y el Load Balancer, se van a crear 2 nuevas instancias sin nombre, las cuales responden las peticiones que le llegan al Load Balancer en su DNS name [LoadbalancerTempimg.amazonaws.com](LoadbalancerTempimg-497736134.us-east-1.elb.amazonaws.com)