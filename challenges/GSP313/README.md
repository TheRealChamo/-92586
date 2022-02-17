# Situación del desafío #
## GSP313 ##

Comenzó una nueva función como ingeniero junior de servicios de nube para Jooli Inc. Se espera que ayude a administrar la infraestructura en Jooli. Entre sus tareas frecuentes se encuentra aprovisionar recursos para los proyectos.  

Se espera que tenga las habilidades y el conocimiento para realizar estas tareas, así que no espere que se le proporcionen guías paso a paso.  

Estas son algunas pautas de Jooli Inc. que debe seguir:  

* Crear todos los recursos en la región o zona predeterminada, a menos que se indique lo contrario
* Normalmente, la asignación de nombres se realiza de la siguiente manera: equipo-recurso; p. ej., una instancia podría llamarse nucleus-webserver1
* Asignar tamaños de recursos rentables (tenga cuidado porque los proyectos se supervisan y el uso excesivo de recursos dará como resultado la finalización del proyecto que los contiene, es decir, posiblemente el suyo; esta es la orientación que el equipo de supervisión está dispuesto a compartir; a menos que se indique algo diferente, utilice f1-micro para VM pequeñas de Linux y n1-standard-1 para Windows u otras aplicaciones, como los nodos de Kubernetes)

### Su desafío ###  

En cuanto se sienta en su escritorio y abre su laptop nueva, recibe varias solicitudes del equipo de Nucleus. Lea cada descripción y, luego, cree los recursos.  

**Tarea 1: Cree una instancia jumphost para el proyecto**  

Utilizaremos esta instancia para realizar el mantenimiento del proyecto.  

Asegúrese de seguir estos pasos:  

- nombrar la instancia nucleus-jumphost
- usar el tipo de máquina de f1-micro
- usar el tipo de imagen predeterminado (Debian Linux)

```zsh
gcloud compute instances create nucleus-jumphost --machine-type=f1-micro --image=debian-10-buster-v20210122
``` 

**Tarea 2: Cree un clúster de servicio de Kubernetes**  

Usted tiene un límite para los recursos que se le permiten crear en su proyecto, si no obtiene el resultado esperado por favor elimine el Clúster antes de crear otro ya que el laboratorio podría cerrarse y usted quedaría suspendido de acceder al mismo.  

El equipo está compilando una aplicación que utilizará un servicio. Este servicio se ejecutará en Kubernetes. Realice lo siguiente:  

- Cree un Clúster (en la Region us-east1) para alojar el servicio.
- Utilice el contenedor hello-app de Docker ('gcr.io/google-samples/hello-app: 2.0') como un marcador de posición. El equipo reemplazará el contenedor con su propio trabajo más tarde.
- Exponga la aplicación en el puerto 8080.

```zsh
gcloud config set compute/zone us-east1-b
```
```zsh
gcloud container clusters create nucleus-jumphost-webserver1
```
```zsh
gcloud container clusters get-credentials nucleus-jumphost-webserver1
```
```zsh
kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0
```
```zsh
kubectl expose deployment hello-app --type=LoadBalancer --port 8080
```
```zsh
kubectl get service
```

**Tarea 3: Configure un balanceador de cargas HTTP**  

Entregaremos el sitio a través de servidores web Nginx, pero queremos asegurarnos de que tenemos un entorno tolerante a errores. Por este motivo, cree un balanceador de cargas HTTP con un grupo de instancias administradas de dos servidores web Nginx. Use el siguiente código para configurar los servidores web. El equipo reemplazará esto con su propia configuración más adelante.  

Usted tiene un límite para los recursos que se le permiten crear en su proyecto, entonces por favor no cree más de dos instancias en su grupo de instancias ya que el laboratorio podría cerrarse y usted quedaría suspendido de acceder al mismo.  

cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

Realice lo siguiente:

- Cree una plantilla de instancias.
- Cree un grupo de destino.
- Cree un grupo de instancias administradas.
- Cree una regla de firewall para permitir el tráfico (80/TCP).
- Cree una verificación de estado.
- Cree un servicio de backend y conecte el grupo de instancias administradas.
- Cree un mapa de URL y un Proxy HTTP de destino para enrutar las solicitudes a su mapa de URL.
- Cree una regla de reenvío.

### 1. Crear el archivo startup.sh con el siguiente contenido ###
```zsh
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```
### 2. Cree una plantilla de instancias ###
```zsh
gcloud compute instance-templates create nginx-template \
--metadata-from-file startup-script=startup.sh
```
### 3. Cree un grupo de destino ###
```zsh
gcloud compute target-pools create nginx-pool
```
Te preguntara la ubicación de la creación del grupo de destino. Si este no es `us-east1` escribe `n` y selecciona el numero correspondiente a `us-east1` en la lista. En mi caso fue el 19. Escribe 19 y presiona Enter.

### 4. Cree un grupo de instancias administradas ###
```zsh
gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool
gcloud compute instances list
```
### 5. Cree una regla de firewall para permitir el tráfico (80/TCP) ###
```zsh
gcloud compute firewall-rules create www-firewall --allow tcp:80
gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool
gcloud compute forwarding-rules list
```
### 6. Cree una verificación de estado ###
```zsh
gcloud compute http-health-checks create http-basic-check
gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80
```
### 7. Cree un servicio de backend y conecte el grupo de instancias administradas ###
```zsh
gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-check --global
gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global
```
### 8. Cree un mapa de URL y un Proxy HTTP de destino para enrutar las solicitudes a su mapa de URL ###
```zsh
gcloud compute url-maps create web-map \
--default-service nginx-backend
gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-map
```
### 9. Cree una regla de reenvío ###
```zsh
gcloud compute forwarding-rules create http-content-rule \
--global \
--target-http-proxy http-lb-proxy \
--ports 80
gcloud compute forwarding-rules list
```