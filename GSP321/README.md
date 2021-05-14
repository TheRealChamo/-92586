# Situación del desafío #
## GSP321 ##

Como ingeniero de la nube de Jooli Inc. recientemente capacitado en Google Cloud y Kubernetes, se le solicitó ayudar a un nuevo equipo (Griffin) a configurar su entorno. El equipo solicitó su ayuda y realizó parte del trabajo, pero necesita que usted lo complete.

Se espera que tenga las habilidades y el conocimiento necesarios para realizar estas tareas, por lo que no recibirá guías paso a paso.

Debe completar las siguientes tareas:

* Crear una VPC de desarrollo con tres subredes de forma manual
* Crear una VPC de producción con tres subredes usando una configuración de Deployment Manager suministrada
* Crear un bastión conectado a ambas VPC
* Crear una instancia de Cloud SQL de desarrollo y conectar y preparar el entorno de WordPress
* Crear un clúster de Kubernetes en la VPC de desarrollo para WordPress
* Preparar el clúster de Kubernetes para el entorno de WordPress
* Crear una implementación de WordPress mediante la configuración suministrada
* Habilitar la supervisión del clúster a través de Stackdriver
* Brindar acceso para un ingeniero adicional  

Estos son algunos estándares de Jooli Inc. con los que debe cumplir:

* Crear todos los recursos en la región us-east1 y en la zona us-east1-b, a menos que se indique lo contrario

* Utilizar las VPC del proyecto

* Normalmente, la asignación de nombres se realiza de la siguiente manera: equipo-recurso; p. ej., una instancia podría llamarse kraken-webserver1

* Asignar tamaños de recursos rentables (tenga cuidado, ya que los proyectos se supervisan y el uso excesivo de recursos dará como resultado la finalización del proyecto que los contiene, es decir, posiblemente el suyo; esta es la orientación que el equipo de supervisión está dispuesto a compartir: a menos que se indique lo contrario, utilice n1-standard-1)

### Su desafío ###

Debe ayudar al equipo con parte del trabajo inicial de un proyecto nuevo. Sus integrantes planean usar WordPress y necesitan que configure un entorno de desarrollo. Si bien parte del trabajo ya está hecho, se requiere su pericia para completar otros aspectos.

En cuanto se siente en su escritorio y abra su laptop nueva, recibirá la siguiente solicitud para completar estas tareas. ¡Buena suerte!

![GSP321-1][GSP321-1]

**Tarea 1: Cree una VPC de desarrollo de forma manual**
Cree una VPC llamada griffin-dev-vpc que solo contenga las siguientes subredes:

`griffin-dev-wp`
Bloque de dirección IP: `192.168.16.0/20`
`griffin-dev-mgmt`
Bloque de dirección IP: `192.168.32.0/20`

```zsh
gcloud compute networks create griffin-dev-vpc --subnet-mode custom
```
```zsh
gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region us-east1 --range=192.168.16.0/20
```
```zsh
gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region us-east1 --range=192.168.32.0/20
```

**Tarea 2: Cree una VPC de producción usando Deployment Manager**
Use Cloud Shell y copie todos los archivos de gs://cloud-training/gsp321/dm.

Verifique la configuración de Deployment Manager y realice los ajustes necesarios. Luego, use la plantilla para crear la VPC de producción con las 2 subredes.

```zsh
gsutil cp -r gs://cloud-training/gsp321/dm .
```
```zsh
cd dm
```
```zsh
sed -i s/SET_REGION/us-east1/g prod-network.yaml
```
```zsh
gcloud deployment-manager deployments create prod-network \
    --config=prod-network.yaml
```
```zsh
cd ..
```

**Tarea 3: Cree un host de bastión**
Cree un host de bastión con dos interfaces de red: una conectada a griffin-dev-mgmt y la otra a griffin-prod-mgmt. Asegúrese de poder acceder al host mediante SSH.

```zsh
gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt  --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=us-east1-b
```
```zsh
gcloud compute firewall-rules create fw-ssh-dev --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-dev-vpc
```
```zsh
gcloud compute firewall-rules create fw-ssh-prod --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-prod-vpc
```

**Tarea 4: Cree y configure una instancia de Cloud SQL****
Cree una instancia de Cloud SQL para MySQL llamada griffin-dev-db en us-east1. Conéctese a la instancia y ejecute los siguientes comandos de SQL para preparar el entorno de WordPress:

```zsh
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;
```

Con estas instrucciones de SQL, se crean la base de datos de WordPress y un usuario con acceso a ella.

Usará el nombre de usuario y la contraseña en la tarea 6.

```zsh
gcloud sql instances create griffin-dev-db --root-password password --region=us-east1
```
```zsh
gcloud sql connect griffin-dev-db
```
```sql
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;
```
```zsh
exit
```

**Tarea 5: Cree un clúster de Kubernetes**
Cree un clúster de 2 nodos (n1-standard-4) llamado griffin-dev en la subred griffin-dev-wp y en la zona us-east1-b.

```zsh
gcloud container clusters create griffin-dev \
  --network griffin-dev-vpc \
  --subnetwork griffin-dev-wp \
  --machine-type n1-standard-4 \
  --num-nodes 2  \
  --zone us-east1-b
```
```zsh
gcloud container clusters get-credentials griffin-dev --zone us-east1-b
```
```zsh
cd ~/
```
```zsh
gsutil cp -r gs://cloud-training/gsp321/wp-k8s .
```

**Tarea 6: Prepare el clúster de Kubernetes**
Use Cloud Shell y copie todos los archivos de gs://cloud-training/gsp321/wp-k8s.

El servidor de WordPress debe acceder a la base de datos de MySQL con el nombre de usuario y la contraseña que creó en la tarea 4. Esto se hace estableciendo los valores como secretos. WordPress también debe almacenar sus archivos de trabajo fuera del contenedor. Por ello, usted debe crear un volumen.

Agregue los siguientes secretos y el volumen al clúster usando wp-env.yaml. Asegúrese de establecer el nombre de usuario en wp_user y la contraseña en stormwind_rules antes de crear la configuración.

También debe proporcionar una clave para una cuenta de servicio ya configurada. Esta cuenta de servicio proporciona acceso a la base de datos para un contenedor de archivo adicional. Use el comando siguiente para crear la clave y luego agréguela al entorno de Kubernetes.

```zsh
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

```zsh
nano wp-k8s/wp-env.yaml
```
`Cambiar usuario y contraseña :`
   `username : wp_user`
   `password : stormwind_rules`
`Guardar`
```zsh
cd wp-k8s
```
```zsh
kubectl create -f wp-env.yaml
```
```zsh
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```
```zsh
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

**Tarea 7: Cree una implementación de WordPress**
Ahora que aprovisionó una base de datos de MySQL y que estableció los secretos y el volumen, podrá crear la implementación usando wp-deployment.yaml. Para crear la implementación, debe editar wp-deployment.yaml y reemplazar YOUR_SQL_INSTANCE por el Nombre de conexión con la instancia de griffin-dev-db. Acceda a su instancia de Cloud SQL para obtener el Nombre de conexión con la instancia.

Después de crear su implementación de WordPress, cree el servicio con wp-service.yaml.

Una vez que se cree el balanceador de cargas, podrá visitar el sitio y asegurarse de que incluye el instalador del sitio de WordPress. En este punto, el equipo de desarrollo tomará el control y completará la instalación, por lo que usted puede pasar a la siguiente tarea.

```zsh
nano wp-deployment.yaml
```
`Reemplazar YOUR_SQL_INSTANCE con el nombre de conexión para la instancia SQL "griffin-dev-db"`
`Guardar`
```zsh
kubectl create -f wp-deployment.yaml
```
```zsh
kubectl create -f wp-service.yaml
```

**Tarea 8: Habilite la supervisión**
Cree una verificación de tiempo de actividad para su sitio de desarrollo de WordPress.

`Desde el Menu:`
`-> Kubernetes Engine -> Services and Ingress -> Copiar la IP del servicio`

`Desde el Menu:`
`-> Monitoring -> Uptime Checks -> + CREATE UPTIME CHECK`
   `Titulo : Wordpress Uptime`
   `Hostname : IP del servicio (192.168.1.1)`
   `Path : /`
`Crear`

**Tarea 9: Brinde acceso para un ingeniero adicional**
Otro ingeniero se sumará al equipo. Le convendrá asegurarse de que tenga acceso al proyecto. Asígnele la función de editor para el proyecto.

La segunda cuenta de usuario para el lab representa al nuevo ingeniero.

`Desde el Menu:`
`-> AIM & Admin -> AIM`    
    `En la barra de busqueda se utiliza el segundo usuario entregado en el inicio del Lab`    
    `Al encontrarlo, debemos ingresar a editar el usuario`    
    `Cambiar el rol de Viewer a Editor`
`Guardar`

[GSP321-1]: /GSP321/GSP321-1.png