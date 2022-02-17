# GSP002 #

# Cómo comenzar a utilizar Cloud Shell y gcloud #

## Tarea 1: Configure el entorno ##

Para verificar cuál es la configuración predeterminada de la región y zona, ejecute los siguientes comandos:

```zsh
gcloud config get-value compute/zone
gcloud config get-value compute/region
```

Si los resultados se muestran como (unset), significa que no se estableció ninguna zona ni región predeterminadas.

- Seleccionar el proyecto en el que vas a trabajar
```zsh
gcloud compute project-info describe --project <your_project_ID>
```
- Configurar variable de entorno `PROJECT_ID`:
```zsh
export PROJECT_ID=<your_project_ID>
```
- Configurar variable de entorno `ZONE`:
```zsh
export ZONE=<your_zone>
```

Instalar una maquina virtual utilizando las variables de entorno:
```zsh
gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone $ZONE
```

## Tarea 2: Instale un componente nuevo ##

- Instale los componentes beta:
```zsh
sudo apt-get install google-cloud-sdk
```
- Habilite el modo gcloud interactive:
```zsh
gcloud beta interactive
```
- Cuando utilice el modo interactivo, presione Tab para completar la ruta de acceso al archivo y los argumentos de recursos. 
```zsh
gcloud compute instances describe <your_vm>
```
**NOTA:** Comience a escribir el siguiente comando y utilice la opción de autocompletado que reemplazará <your_vm> por una VM existente de su proyecto

## Tarea 3: Conéctese a su instancia de VM con SSH ##

Para conectarse a su VM con SSH, ejecute el siguiente comando:
```zsh
gcloud compute ssh gcelab2 --zone $ZONE
```