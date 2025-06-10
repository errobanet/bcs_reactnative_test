id: bcs-face-reactnative

# Como usar BCS en REACT-NATIVE

## Introducción

En la siguiente guía vamos a ver como integrar el módulo de verificación de identidad de BCS en React-native.

Para poder hacer esto vas a necesitar tener que tener configurado tu entorno para desarrollo.

Podes itilizar cualquier IDE, vamos a hacer todo x linea de comandos, para la versión de iOS es necesaria una Mac y XCode.


## Plugin

Para poder usar BCS en tu proyecto de React-native necesitas ejecutar los siguientes comando (utilizamos yarn, pero podes usar npm):

```
yarn add @erroba/react-native-bcssdk
```

Adicionalmente necesitamos agregar unas dependencias nativas que no pueden agregarse en el plugin.

* Android: archivo bcssdk.1.x.x.aar
* iOS: carpeta bcssdk con dependencias

Los mismos son provistos al equipo de desarrollo por erroba.

## Permisos
La verificación de identidad con rostro necesita algunos permisos en el dispositivo:
* Cámara
* Micrófono (solo android)

El SDK los solicita, pero si son denegados obtendras la respuesta `PERMISSIONS_ERROR`.
<aside class="negative">
Si los permisos son denegados multiples veces la aplicación ya no mostrara la solicitud, este es el comportamiento natural tanto en Android como en iOS, una práctica habitual es abrir la configuración de la APP para que el usuario lo habilite.
</aside>

### Android

En Android no necesitas editar el archivo AndroidManifest.xml, el plugin agrega lo necesario.

### iOS

En iOS hay que hacer unos ajustes en el proyecto APP de  iOS.

Debes agregar la entrada de `NSCameraUsageDescription` a tu `Info.plist`

```xml
<key>NSCameraUsageDescription</key>
<string>Es necesario el uso de la cámara.</string>
```

## Librerias nativas - Android

Primero vamos a copiar algunas librerias nativas que son necesarias que funcione el proyecto.

1. Copia el archivo `bcssdk-x.x.x.aar` a la carpeta `android/app/libs`
2. Edita el archivo `android/app/build.gradle` y agrega la dependencia `implementation files('libs/bcssdk-1.x.x.aar')`:

Debería quedarte algo así:

![image_caption](img/android_rn.png)

## Librerias nativas - iOS

Para iOS es necesario agregar los recursos de la carpeta bcssdk al proyecto APP de Xcode.

1. Copia la carpeta `bcssdk` que esta dentro de iOS en el zip de SDK a la carpeta `ios/` de tu proyecto de react.
2. Edita el `Podfile` y agrega `pod 'bcssdk', :path => 'bcssdk'`

![image_caption](img/ios_ref01_rn.png)

3. Ejecuta `pod install` para que se instalen las dependencias

```
pod install
```

## Utilización del cliente

A continuacion vamos a mostrar el uso suponiendo que ya tenemos un código de transacción para verificar.

### Verificación

Ya tenemos todo configurado, vamos a usar el cliente!

Para llamar a la verificación solo tenes que hacer los imports y llamar al método `faceVerify`
```typescript
import { faceVerify, setColors, setUrlService } from '@erroba/react-native-bcssdk';

async processVerify(code) {
    return await faceVerify(code);
}
```

### Respuestas

La respuesta de la llamada a `faceVerify` es una enumeración de typescript y puede ser uno de los siguientes valores:

* DONE
* CANCELED
* PERMISSIONS_ERROR
* CONNECTION_ERROR
* TRANSACTION_NOT_FOUND


> Según la respuesta obtenida es la acción que debes realizar en tu app.

#### DONE

La operación finalizó en el servidor, debes obtener el resultado desde tu backend, puede ser tanto Verificado como NO verificado.

#### CANCELED

El usuario canceló la operación, generalmente con la opción o gesto de volver.

#### PERMISSIONS_ERROR

Esta respuesta se da cuando no hay permisos para cámara y microfono, debes haberlos solicitado antes y verificarlos.

#### CONNECTION_ERROR

No fue posible conectar con los servidores de BCS, puede deberse a un problema de conectividad entre el dispositivo e internet/servidores de BCS.

#### TRANSACTION_NOT_FOUND

No se encontró la transacción x el identificador `code`. Ten en cuenta que después de creada solo puede ser procesada en un período corto de tiempo.

## Estilos

La interfaz de la verificación es minimalista, el único control de interacción con el usuario es un botón para `Reintentar` la operación.

Podes establecer los colores para los controles llamando a la función `setColors` del plugin.

```typescript
import { setColors } from '@erroba/react-native-bcssdk';

async initializePluginColors() {
    await setColors('#6200ea', '#ffffff');
}
```

## Ambiente QA/Docker

Por defecto el cliente utiliza el ambiente productivo. Si deseas usar al ambiente de calidad o desarrollo con docker podes cambiar la URL de los servicios.

Para hacerlo está disponible la función `setUrlService` en la api.

```typescript
import { setUrlService } from '@erroba/react-native-bcssdk';

async initializeUrl() {
    await setUrlService(url);
}
```

> No dejes este código en el RELEASE de tu aplicación.

## Servicio BCS

Para utilizar la verificación, previamente debes haber generado un código de transacción desde el backend de tu aplicación.

![image_caption](img/app_seq.png)


>Es recomendable NO exponer en tus APIS la identificación de la persona, sino hacerlo sobre algún identificador de onboarding o transacción. De esta froma podés prevenir el uso de tu API por terceros.


