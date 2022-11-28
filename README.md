# Chat en Ionic con autenticación en Firebase.

## Prerequisitos
Se deben instalar las siguientes librerias en el directorio del proyecto:
- ```npm install @ionic-native/camera```
- ```npm install @angular/fire```
- ```npm install @ionic/angular```
- ```npm install crypto-js```

COMPONENTES.

- Página de Registro.
Se crea un formulario con email y contraseña con las validaciones correspondientes.
  
- Inicio de Sesión.
Para el inicio de sesión se crea un nuevo formulario con los mismos campos y validaciones que el registro.
La función loginUser permite iniciar sesión con el método loginUser proporcionado por AuthenticateService.
  
- Chat.
Para el chat en firebase.service se crean dos funciones para interactuar con los datos de firebase, una para almacenar y otra para leer los datos.
En el caso de almacenar se utiliza la función sendMessage con ayuda de AngularFireDatabase, mientras que para obtener los datos se utiliza la función getMessages.
 
Para tomar foto se utiliza la función TakePhoto que va a permitir almacenar la imágen en el storage de firebase y se va a obtener la url de la misma para almacenarla en la base de datos.

```
takePhoto( sourceType ) {
    try {
      const options: CameraOptions = {
        quality: 50,
        targetHeight: 600,
        targetWidth: 600,
        destinationType: this.camera.DestinationType.DATA_URL,
        encodingType: this.camera.EncodingType.JPEG,
        mediaType: this.camera.MediaType.PICTURE,
        correctOrientation: true,
        sourceType
      };

      this.camera.getPicture( options )
        .then( async ( imageData ) => {
          console.log( 'IMAGE DATA', imageData );
          this.tmpImage = 'data:image/jpeg;base64,' + imageData;
          const putPictures = firebase.storage().ref( 'messages/' + this.imageId + '.jpeg' );
          putPictures.putString( this.tmpImage, 'data_url' ).then( ( snapshot ) => {
            console.log( 'snapshot', snapshot.ref );
          } );
          const getPicture = firebase.storage().ref( 'messages/' + this.imageId + '.jpeg' ).getDownloadURL();
          getPicture.then( ( url ) => {
            this.message = url;
          } );
        } )
        .catch( ( e ) => {
          console.log( e );
          this.tmpImage = undefined;
        } );
    } catch ( e ) {
      console.log( e );
      this.tmpImage = undefined;
    }
  }
  ```

Finalmente, la función para enviar el mensaje sendMessage que va a permitir almacenar los datos en un arreglo, con el mensaje o imagen encriptado dependiendo del caso y va a utilizar la función creada en firebase.service para guardar los datos en la base de datos.

```
async sendMessage() {
    let chat = {};
    this.encryptText( this.message );
    if ( this.tmpImage !== undefined ) {
      chat = {
        uid: this.id,
        email: this.userEmail,
        image: this.encryptedText,
        date: Date.now()
      };
    } else {
      chat = {
        uid: this.id,
        email: this.userEmail,
        message: this.encryptedText,
        date: Date.now()
      };
    }

    try {
      await this.chatService.sendMessage( chat );
      this.message = '';
    } catch ( e ) {
      console.log( 'error', e );
    }
  }
  ```
 
IMAGENES SEGURAS.

En el storage de firebase se agregan reglas para leer y escribir en donde el usuario que realiza las peticiones deberá estar autenticado y la imagen debe tener el formate image/jpeg.

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /messages/{imageId=**} {
      allow read, write: if request.auth !== null;
      allow write: if resource.contentType('image/jpeg')
    }
  }
}
```

## POR ULTIMO PARA PROBAR EN WEB:
- ```npm i cordova```
- ```ionic cordova platform add browser```
- ```ionic cordova run browser```



