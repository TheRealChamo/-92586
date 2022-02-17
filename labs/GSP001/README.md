# GSP001 #

# Cómo crear una máquina virtual #

## Creación de Maquina Virtual en Google Cloud ##

```zsh
gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-f
```
**NOTA:**  
`gcelab2` : Nombre de la Maquina Virtual  
`--machine-type n1-standard-2` : Tipo de Maquina **n1-standard-2**  
`--zone us-central1-f` : Zona **us-central1-f**  

## Conectarse a la Maquina virtual por SSH ##

```zsh
gcloud compute ssh gcelab2 --zone us-central1-f
```

## Instalar NGINX ##

- Entrar en modo superuser
```zsh
sudo su -
```
- Actualización de Ficheros y Librerias
```zsh
apt-get update
```
- Instalación del Paquete NGINX
```zsh
apt-get install nginx -y
```
- Ver estatus
```zsh
ps auwx | grep nginx
```