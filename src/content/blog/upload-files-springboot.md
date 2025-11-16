--- 
title: "Upload files with Spring Boot / Subir imagenes haciendo uso de SpringBoot" 
description: "Learn how to upload files with Spring Boot / Aprende a subir archivos con Spring Boot"
pubDate: "Nov 15, 2025"
---
# Subir archivos con Spring Boot. (En el servidor)
En esta guía voy a tratar de explicar cómo se podría subir un archivo al servidor haciendo uso de Spring Boot. En este caso, subiremos imágenes con formato JPG, PNG, JPEG.

## Requisitos 
- Java 17
- Spring Boot 3.0.0
- Maven 3.8.6
- IDE IntelliJ IDEA

En este caso, la dependencia que utilizaremos será la de Spring Web. Para ello, debemos añadir la siguiente dependencia en el archivo pom.xml:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
``` 

## Pasos para subir archivos
Voy a lograr hacerlo de manera ordenada con una arquitectura simple. Primeramente, el que recibirá la petición que será el controlador y los servicios que se encargarán de la lógica de está aplicación. En servicio tendremos una interfaz la cual definirá el método y su correspondiente implementación.

### 1. Creación de la interfaz. 
Crearemos una interfaz llamada `uploadFile` que contendrá el método que recibirá la petición y que se 
encargará de subir el archivo. Está interfaz será la que se inyectará en el controlador.

```java
package com.doposama.uploadFile.service;

import org.springframework.web.multipart.MultipartFile;

public interface IUploadFile {

    String uploadFile(MultipartFile file) throws Exception;
}
```

Como se puede apreciar, la interfaz recibe un parámetro de tipo `MultipartFile` que representa el archivo que se va a subir. Además de esto, vemos que se encuentra en el package de la interfaz `com.group.uploadFile.service`.

### 2. Creación de la implementación de la interfaz. 
Crearemos una clase llamada `UploadFileServiceImpl` que implementará la interfaz `IUploadFile`. Está clase se encargará de realizar toda la lógica del procesamiento para subir el archivo, en este caso va a verificar que el archivo sea de tipo JPG, PNG, JPEG y que el tamaño del archivo no exceda el máximo permitido. Antes de esto, me gustaría explicar que en el application.properties también defini el tamaño máximo de archivo que se va a subir.


##### application.properties
```java
spring.application.name=uploadFile

spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

Como se puede apreciar, en este caso, el tamaño máximo permitido para subir un archivo es de 10MB. Ahora en la implementación: 


### uploadFileServiceImpl.java
```java 
package com.doposama.uploadFile.service.implementation;

import com.doposama.uploadFile.service.IUploadFile;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.UUID;

@Service
public class UploadFilesImpl implements IUploadFile {

    @Override
    public String uploadFile(MultipartFile file) throws Exception {
        try{
            String randomName = UUID.randomUUID().toString();
            String originalFileName = file.getOriginalFilename();
            String nameWithoutExt = originalFileName.substring(0, originalFileName.lastIndexOf("."));

            String ext = originalFileName.substring(originalFileName.lastIndexOf("."));
            String newName = nameWithoutExt + "_" + randomName + ext;

            byte[] fileBytes = file.getBytes();

            Long fileSize = file.getSize();
            Long maxFileSize = (long) (10 * 1024 * 1024);

            if(fileSize > maxFileSize){
                return "File size must be less than or equal 10MB, check your file";
            }

            System.out.println(newName + " Extención: " + ext);

            if(!ext.equalsIgnoreCase(".png") &&
                !ext.equalsIgnoreCase(".jpeg") &&
                !ext.equalsIgnoreCase(".jpg")
            ){
                return "Only accepted file formats: JPEG, PNG, and JPG.";
            }

            File folder = new File("src/main/resources/upload");

            if(!folder.exists()){
                folder.mkdirs();
            }

            Path path = Paths.get("src/main/resources/upload/" + newName);

            Files.write(path, fileBytes);

            return "File Upload successfull";

        }catch (Exception e){
            throw new Exception(e.getMessage());
        }
    }
}
```
Para los que no entiendan los parametros de la interfaz, el multipartFile es un objeto que representa el archivo que se va a subir, este objeto tiene métodos como getOriginalFilename(), getSize(), getBytes(), etc. Viene incluido en la dependencia de spring-boot-starter-web.

### UploadFileController.java
Finalmente, vamos a crear el controlador que recibirá la petición y que se encargará de llamar a la interfaz y de devolver el resultado de la petición. Recordar que hay que inyectar la dependecia de la interfaz en el controlador. Cuestión de patrones de diseño, singleton de dicha interfaz. 

```java
package com.doposama.uploadFile.controller;

import com.doposama.uploadFile.service.IUploadFile;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/file")
public class UploadFilesController {

    @Autowired
    IUploadFile uploadFileService;

    @PostMapping("/uploadFile")
    private ResponseEntity<?> uploadFile(@RequestParam("file") MultipartFile file) throws Exception {
        try{
            Object response = uploadFileService.uploadFile(file);
            return new ResponseEntity<>(response, HttpStatus.ACCEPTED);
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("Error al subir el archivo: " + e.getMessage());
        }
    }
}
```

### Pruebas
Para probar la aplicación, vamos a utilizar Postman. 
Antes de eso, inicializamos el proyecto con maven. 
![terminal](/captura_terminal.png)

Una vez que se inicializa el proyecto, vamos a subir el archivo que queremos subir. 
![postman](/captura_postman.png)

Si todo va bien, obtendremos un mensaje de éxito.

Ahora revisando el servidor la carpeta `src/main/resources/upload` debería tener el archivo que subimos.
![servidor](/captura_servidor.png)

### Conclusión

En este tutorial, aprendimos a subir archivos a un servidor haciendo uso de Spring Boot. Aprendimos a utilizar la interfaz de Spring Web para recibir la petición y la implementación de la misma para realizar la lógica de procesamiento.