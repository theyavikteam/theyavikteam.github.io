---
layout: post
title: Migración Realm en Android con Kotlin
---

![Android Kotlin](https://codigoonclick.com/wp-content/uploads/2018/03/kotlin-con-android-caracteristicas.jpeg)

subtitle: 0 - Aterricemos

Mi primera entrada en el blog va sobre un caso que me ocurrió la semana pasada antes de *migrar* al pueblo a disfrutar de unos días de descanso. 
   
A lo largo del desarrollo de una app nos llevamos gran parte del tiempo arrancando nuestros proyectos una y otra vez con una alegría desmedida (tu vida es eso que pasa mientras se construye el apk...) en el emulador o en un dispositivo real para hacer pruebas funcionales, depurar errores o simplemente ver las maquetaciones de las vistas.

   Es así incluso cuando trabajamos en apps en las que intervienen bases de datos, puesto que durante el desarrollo podemos cambiar libremente su esquema (si borras la base de datos no hay problema) hasta que estos proyectos pasan a un entorno de validación o más crítico si cabe pasan al mundo real. Hablemos pues de las migraciones. Más concretamente a cómo migrar una base de datos implementada con [Realm](https://realm.io/) en una aplicación Android.
   
   ![Realm Logo](https://realm.io/assets/img/social/realmDark.jpg)
   
## 1 - Configurar Realm, crear nuestra primera tabla y entrada en base de datos

Empezamos creando un proyecto de cero con Android Studio y habilitando la compatibitilidad con Kotlin. Una vez creado pasamos a integrar la dependencia de Realm. Primero añadimos el plugin con la versión más actual en el archivo build.gradle del proyecto.

{{ "{% highlight groovy " }}%}  
classpath "io.realm:realm-gradle-plugin:5.2.0"
{{ "{% endhighlight " }}%}  

Y a continuación en el fichero build.gradle del módulo de la aplicación aplicamos dicho plugin.

{{ "{% highlight groovy " }}%}  
apply plugin: 'realm-android'
{{ "{% endhighlight " }}%} 

## 2 - Vaya, se me ha olvidado un parámetro de Persona

## 3 - Además de personas, los perros tienen derecho a existir

## 4 - ¿Y si las personas tienen un perro de mascota?

## 5 - Conclusión


