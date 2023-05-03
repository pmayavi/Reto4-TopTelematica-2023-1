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
Reinciar la conexion co la maquina, y se toma la imagen de moodle de https://hub.docker.com/r/bitnami/moodle:
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
```sh
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-0032ec83d2f306181.efs.us-east-1.amazonaws.com:/ /moodledata
```
```sh
sudo nano /etc/fstab
```
En el archivo ingresar en la ultima linea:
```sh
fs-0032ec83d2f306181.efs.us-east-1.amazonaws.com:/ /moodledata nfs nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport
```
Y acabar con los comandos:
```sh
mount -a
mount | grep nfs
```
Ahora en AWS se detiene la maquina y se crea su imagen
Se crea un template nuevo desde esta imagen
