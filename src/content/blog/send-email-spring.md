---
title: "Send Email with Spring | Enviar correo con Spring"
description: "Learn how to send email with Spring | Aprende a enviar correo con Spring"
pubDate: "Nov 19, 2025"
githubUrl: "https://github.com/DopoDev/send_email_spring"
---

# Enviar correo con Spring
El envío de correos es una tarea muy usual en las aplicaciones empresariales o personales, motivo de notificación o recordatorio, inclusive utilizada para informar en masas a usuarios sobre un evento o un cambio en la aplicación. \
La idea presente es mostrar un pequeño y simple endpoint el cual se encargará de enviar el correo a los usuarios, o en este caso a un unico usuario.

## Requisitos
- Java 17
- Spring Boot 3.0.0
- Maven 3.8.6
- IDE IntelliJ IDEA

En este caso, la dependencia que utilizaremos será la de Spring Web. También utilizaremos la de Spring Mail para enviar el correo. Para ello, debemos añadir la siguiente dependencia en el archivo pom.xml:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```
Además de esta dependencia, crearemos un pequeño template de correo HTML para que el correo se vea correctamente en el cliente. Para esto añadiremos también la dependencia de Thymeleaf.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## Pasos para enviar correo 
### 1. Crear el archivo de configuración para el correo. 
En este archivo especificaremos las propiedades necesarias para realizar la conexión con la cuenta de correo electrónico, en este caso utilizaremos Gmail. Debemos tener en cuenta que ahora Gmail requiere de una contraseña para poder enviar correos, por lo que debemos añadir la contraseña en el archivo de configuración (Esta contraseña debe ser generada en la cuenta de Gmail).

```properties
email.username = correoprueba1234@gmail.com
email.password = gddsa earc isoy ftko
```
Obviamente estos datos son completamente ficticios, por lo que debemos añadir nuestros propios datos, pero funcionan como ejemplo de configuración.
Cabe destacar que este archivo properties es diferente al que genera Sprig Boot, "application.properties", prefiero almacenarlo en un archivo independiente que llame email.properties.

### 2. Creación del Archivo emailDto.java
Crearemos un archivo llamado emailDto.java que contendrá el modelo que se utilizará para enviar el correo. Este archivo debe tener la siguiente estructura:

```java
package com.johao.mail.service.model;

public class EmailDTO {

    private String destinatario;
    private String asunto;
    private String mensaje;

    public String getDestinatario() {
        return destinatario;
    }

    public void setDestinatario(String destinatario) {
        this.destinatario = destinatario;
    }

    public String getAsunto() {
        return asunto;
    }

    public void setAsunto(String asunto) {
        this.asunto = asunto;
    }

    public String getMensaje() {
        return mensaje;
    }

    public void setMensaje(String mensaje) {
        this.mensaje = mensaje;
    }
}
```
El correo que se recibirá mediante el controlador tendrá simplemente un mensaje, el asunto y el destinatario. Nada complejo.

### 3. Archivo de configuración del Correo
Este será el archivo con más complejidad, donde se utilizará la configuración de email definida anteriormente. Para esto, se usa el decorador @PropertySource para que el archivo de configuración sea cargado desde el archivo email.properties, debemos añadir el classpath para este decorador. También debemos agregar el decorador de @Configuration para que este archivo sea reconocido como configuración.

```java
package com.johao.mail.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.io.DefaultResourceLoader;
import org.springframework.core.io.ResourceLoader;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.JavaMailSenderImpl;

import java.util.Properties;

@Configuration //Indicar que es una configuración
@PropertySource("classpath:email.properties")
public class EmailConfig {

    @Value("${email.username}")
    private String email;

    @Value("${email.password}")
    private String password;

    private Properties getMailProperties(){
        Properties properties = new Properties();
        properties.put("mail.smtp.auth", "true");
        properties.put("mail.smtp.starttls.enable", true);
        properties.put("mail.smtp.host", "smtp.gmail.com");
        properties.put("mail.smtp.port", "587");

        return properties;
    }

    @Bean
    public JavaMailSender javaMailSender(){
        JavaMailSenderImpl javaMailSender = new JavaMailSenderImpl();

        javaMailSender.setJavaMailProperties(getMailProperties());
        javaMailSender.setUsername(email);
        javaMailSender.setPassword(password);

        return javaMailSender;
    }

    @Bean
    public ResourceLoader resourceLoader(){
        return new DefaultResourceLoader();
    }
}
```

Como pueden ver, en los parametros de email y password, le asignamos los valores de la configuración definida en el archivo email.properties.
Creamos un objeto Properties para almacenar y obtener las propiedades de la conexión con el servidor SMTP de Gmail. Esto devuelve un objeto que: 
- **mail.smtp.auth = true** -> Esto permite la autenticación de usuario y contraseña.
- **mail.stmp.starttls.enable = true** -> Esto permite que el servidor SMTP de Gmail se comunique utilizando TLS para una conexión segura.
- **mail.smtp.host = smtp.gmail.com** -> Esto indica que el servidor SMTP de Gmail es smtp.gmail.com
- **mail.smtp.port = 587** -> Esto indica que el puerto de conexión es 587

#### Bean principal JavaMailSender
Este bean es el encargado de enviar los emails. ¿Que hace este bean? se encargará de crear un objeto para el JavaMailSenderImpl (Proximamente veremos este objeto). Pondrá las propiedades que definimos de SMTP, conecta al servidor SMTP de Gmail y conectará con el usuario y contraseña que definamos en el archivo email.properties. Además de esto, registrará el objeto como un bean disponible para la inyección de dependencias.

#### Bean ResourceLoader
Este bean es el encargado de cargar los archivos de configuración. En este caso, será de utilidad para cargar archivos estaticos, HTML, imágenes, etc.

### 4. Creación de la interfaz IEmailService
Crearemos una interfaz IEmailService que contendrá el método que recibirá la petición y que se encargará de enviar el correo. Está interfaz será la que se inyectará en el controlador.

```java
package com.johao.mail.service;

import com.johao.mail.service.model.EmailDTO;
import jakarta.mail.MessagingException;

public interface IEmailService {

    public void enviarCorreo(EmailDTO emailDTO) throws MessagingException;
}
```
En este caso, la función de enviar correo recibe un parámetro de tipo EmailDTO que representa el modelo que se utilizará para enviar el correo. No devolverá nada, solo enviará el correo.

### 5. Creación de la implementación de la interfaz IEmailService
En esta clase implementaremos la interfaz IEmailService. Esta clase se encargará de realizar toda la lógica del procesamiento para enviar el correo, en este caso va a verificar que el destinatario, asunto y mensaje. 
```java
package com.johao.mail.service.impl;

import com.johao.mail.service.IEmailService;
import com.johao.mail.service.model.EmailDTO;
import jakarta.mail.MessagingException;
import jakarta.mail.internet.MimeMessage;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@Service
public class EmailServiceImpl implements IEmailService {

    @Autowired
    TemplateEngine templateEngine;

    @Autowired
    JavaMailSender javaMailSender;

    @Override
    public void enviarCorreo(EmailDTO emailDTO) throws MessagingException {
        try {


            MimeMessage message = javaMailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");

            helper.setTo(emailDTO.getDestinatario());
            helper.setSubject(emailDTO.getAsunto());

            // helper.setText("Hola correo"); // Para enviar texto plano por el mensaje

            Context context = new Context();
            context.setVariable("mensaje", emailDTO.getMensaje());

            String contenidoHTML = templateEngine.process("email", context);

            helper.setText(contenidoHTML, true);
            javaMailSender.send(message);
        } catch (RuntimeException e) {
            throw new RuntimeException("Error al enviar el correo: " + e.getMessage());
        }
    }
}
```
Se puede apreciar que hacemos inyección de dependencias de JavaMailSender y TemplateEngine. JavaMailSender es el componente configurado en emailConfig.java para conectarse con el servidor SMTP de Gmail. TemplateEngine es el decorador que utilizaremos para procesar el HTML.

Definimos el método enviarCorreo que recibe un parámetro de tipo EmailDTO. En este caso cualquier error arrojará una excepción de tipo RuntimeException. Esté DTO recibirá los parametros de destinatario, asunto y mensaje.

Para preparar y crear el mensaje utilizaremos la clase MimeMessage, el cual permite correos HTML, arhivos adjuntos, imágenes, etc. Para esto crearemos un objeto MimeMessage y un objeto MimeMessageHelper que se encargará de preparar el mensaje.
MimeMessageHelper facilitará la configuración del correo. En este caso, agregamos los parametros del message, creado en MimeMessage, true para que el mensaje sea HTML y archivos adjuntos y UTF-8 como codificación.

Para la configuración del destinatario y el asunto utilizamos el método setTo y setSubject respectivamente. Insertando los valores traidos del DTO del email.

Ya finalmente, solo nos queda preparar la platilla con thymeleaf y enviar el correo. Para esto, utilizaremos el objeto Context que se encargará de insertar los valores del DTO en la platilla. En este caso, solo se inserta el mensaje. 

Posteriormente creamos un objeto TemplateEngine que se encargará de procesar la platilla HTML.

### 6. Creación del Template HTML para el correo
Crearemos un archivo llamado email.html que contendrá la platilla HTML para el correo. Este archivo debe tener la siguiente estructura:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>DopoDev - Notificación</title>
</head>
<body style="margin:0; padding:0; background-color:#f5f6fa; font-family:Arial, Helvetica, sans-serif;">

<!-- CONTENEDOR PRINCIPAL -->
<table width="100%" border="0" cellspacing="0" cellpadding="0"
       style="background-color:#f5f6fa; padding: 40px 0;">
    <tr>
        <td align="center">

            <!-- TARJETA -->
            <table width="600" cellpadding="0" cellspacing="0"
                   style="background-color:#ffffff; border-radius:8px;
                              box-shadow:0 4px 10px rgba(0,0,0,0.08);
                              overflow:hidden;">

                <!-- ENCABEZADO -->
                <tr>
                    <td style="background-color:#1B263B; padding:30px; text-align:center;">
                        <h1 style="color:white; margin:0; font-size:28px; letter-spacing:1px;">
                            DopoDev
                        </h1>
                        <p style="color:#dfe6ed; margin:10px 0 0; font-size:14px;">
                            Innovación · Desarrollo · Tecnología
                        </p>
                    </td>
                </tr>

                <!-- MENSAJE PRINCIPAL -->
                <tr>
                    <td style="padding: 35px 40px; text-align:left;">
                        <h2 style="color:#1B263B; font-size:22px; margin-top:0;">
                            ¡Hola!
                        </h2>

                        <p style="color:#555; line-height:1.6; font-size:16px; margin-bottom:25px;"
                           th:text="${mensaje}">
                        </p>

                        <p style="color:#555; font-size:15px; line-height:1.6;">
                            Si tienes alguna duda o necesitas soporte, no dudes en
                            contactarnos. Nuestro equipo estará encantado de ayudarte.
                        </p>
                    </td>
                </tr>

                <!-- PIE DE PÁGINA -->
                <tr>
                    <td style="background-color:#1B263B; padding:20px; text-align:center;">
                        <p style="color:#dfe6ed; font-size:13px; margin:0;">
                            © 2025 DopoDev. Todos los derechos reservados.
                        </p>
                    </td>
                </tr>

            </table>
            <!-- FIN TARJETA -->

        </td>
    </tr>
</table>

</body>
</html>
```

### 7. Creación del Controlador
Crearemos un controlador que recibirá la petición y que se encargará de llamar a la interfaz y de devolver el resultado de la petición. Recordar que hay que inyectar la dependencia de la interfaz en el controlador. Cuestión de patrones de diseño, singleton de dicha interfaz.

```java
package com.johao.mail.controller;

import com.johao.mail.service.IEmailService;
import com.johao.mail.service.model.EmailDTO;
import jakarta.mail.MessagingException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.HttpStatusCode;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/email")
public class EmailController {

    @Autowired
    public IEmailService emailService;

    @PostMapping("/send")
    public ResponseEntity<String> enviarCorreo(@RequestBody EmailDTO email) throws MessagingException {
        emailService.enviarCorreo(email);
        return new ResponseEntity<>("Correo enviado, revisa la bandeja de entrada", HttpStatus.OK);
    }
}
```
