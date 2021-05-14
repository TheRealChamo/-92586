# Situación del desafío #
## GSP315 ##

Recién comienza a desempeñarse en su función como ingeniero junior de servicios de nube para Jooli Inc. Hasta ahora, estuvo ayudando a los equipos a crear y administrar recursos de Google Cloud Platform.

Se espera que tenga las habilidades y el conocimiento para realizar estas tareas, por lo que no recibirá guías paso a paso.

### Su desafío ###

Se le solicitó ayudar a un equipo de desarrollo recién formado con el trabajo inicial de un nuevo proyecto, llamado Recuerdos, el cual consiste en almacenar y organizar fotos. Se le pidió que asista al equipo de Recuerdos con la configuración inicial del entorno de desarrollo de la aplicación. Recibe una solicitud en la que se le indica que debe completar las siguientes tareas:

* Crear un depósito para almacenar fotos
* Crear un tema de Pub/Sub para que lo utilice la Cloud Function que usted genere
* Crear una Cloud Function
* Quitar el acceso del ingeniero anterior de servicios de nube para que ya no pueda ingresar al proyecto Recuerdos

Estas son algunas pautas de Jooli Inc. que debe seguir:

* Crear todos los recursos en la región us-east1 y en la zona us-east1-b, a menos que se indique lo contrario
* Utilizar las VPC del proyecto
* Normalmente, la asignación de nombres se realiza de la siguiente manera: equipo-recurso; p. ej., una instancia podría llamarse kraken-webserver1
* Asignar tamaños de recursos rentables. Tenga cuidado porque los proyectos se supervisan y el uso excesivo de recursos dará como resultado la finalización del proyecto que los contiene, es decir, posiblemente el suyo. Esta es la orientación que el equipo de supervisión está dispuesto a compartir: a menos que se indique lo contrario, utilice f1-micro para VM pequeñas de Linux y n1-standard-1 para Windows o demás aplicaciones, como los nodos de Kubernetes

A continuación, se describe cada tarea en detalle. ¡Buena suerte!

```zsh
gcloud config set compute/region us-east1
gcloud config set compute/zone us-east1-b
```

**Tarea 1: Cree un depósito**
Debe crear un depósito para almacenar las fotos.

```zsh
gsutil mb -b on -l us-east1 gs://REEMPLAZAR_POR_ID_DEL_PROYECTO
```

**Tarea 2: Cree un tema de Pub/Sub**
Cree un tema de Pub/Sub para que la Cloud Function envíe mensajes.

```zsh
gcloud pubsub topics create my-topic
```

**Tarea 3: Cree la Cloud Function de la miniatura**
Cree una Cloud Function que se ejecute cada vez que se agregue un objeto al depósito configurado en la tarea 1. La función se escribe en Node.js 10. Asegúrese de configurar la Función que se ejecutará en thumbnail.

En la línea 15 de index.js, reemplace el texto REPLACE_WITH_YOUR_TOPIC con el tema creado en la tarea 2.

index.js:

```js
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const gcs = require("@google-cloud/storage")();
const PubSub = require("@google-cloud/pubsub");
const imagemagick = require("imagemagick-stream");

exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "REPLACE_WITH_YOUR_TOPIC";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });

          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```
package.json:
```js
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/storage": "1.5.1",
    "@google-cloud/pubsub": "^0.18.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```

Debe subir una imagen JPG o PNG al depósito y, luego, verificaremos que se haya creado la miniatura. Utilice cualquier imagen JPG o PNG, o bien esta imagen https://storage.googleapis.com/cloud-training/gsp315/map.jpg, que debe descargar en su máquina para luego subirla al depósito. Al cabo de unos instantes, aparecerá una imagen en miniatura (seleccione ACTUALIZAR DEPÓSITO).

**Tarea 4: Quite al ingeniero anterior de servicios de nube**
Verá que hay dos usuarios: uno corresponde a su cuenta (con la función Propietario) y el otro, al ingeniero anterior de servicios de nube (con la función Visualizador). Nos gusta mantener normas de seguridad estrictas, así que quite el acceso del ingeniero anterior de servicios de nube para que ya no pueda ingresar al proyecto.

```zsh
gcloud projects remove-iam-policy-binding qwiklabs-gcp-03-0ff7abf92fc4\
--member=user:student-03-9389d0c3a86b@qwiklabs.net --role=roles/viewer
```