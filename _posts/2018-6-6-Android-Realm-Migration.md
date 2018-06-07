---
layout: post
title: Migración Realm en Android con Kotlin
---

![Android Kotlin](https://codigoonclick.com/wp-content/uploads/2018/03/kotlin-con-android-caracteristicas.jpeg)

## 0 - Aterricemos

Mi primera entrada en el blog va sobre un caso que me ocurrió la semana pasada antes de *migrar* al pueblo a disfrutar de unos días de descanso. 
   
A lo largo del desarrollo de una app nos llevamos gran parte del tiempo arrancando nuestros proyectos una y otra vez con una alegría desmedida (tu vida es eso que pasa mientras se construye el apk...) en el emulador o en un dispositivo real para hacer pruebas funcionales, depurar errores o simplemente ver las maquetaciones de las vistas.

   Es así incluso cuando trabajamos en apps en las que intervienen bases de datos, puesto que durante el desarrollo podemos cambiar libremente su esquema (si borras la base de datos no hay problema) hasta que estos proyectos pasan a un entorno de validación o más crítico si cabe pasan al mundo real. Hablemos pues de las migraciones. Más concretamente a cómo migrar una base de datos implementada con [Realm](https://realm.io/) en una aplicación Android.
   
   ![Realm Logo](https://realm.io/assets/img/social/realmDark.jpg)
   
## 1 - Configurar Realm, crear nuestra primera tabla y entrada en base de datos

Empezamos creando un proyecto de cero con Android Studio y habilitando la compatibitilidad con Kotlin. Una vez creado pasamos a integrar la dependencia de Realm. Primero añadimos el plugin con la versión más actual en el archivo *build.gradle* del proyecto:

	<pre><code class="groovy">classpath "io.realm:realm-gradle-plugin:5.2.0"
	</code></pre>

Y a continuación en el fichero *build.gradle* del módulo de la aplicación aplicamos dicho plugin:

<pre><code class="groovy">apply plugin: 'realm-android'
</code></pre>

Una vez tenemos la librería en nuestro proyecto hemos de configurar la instancia de Realm con la que vamos a trabajar. Para ello creamos una clase que extienda de Application y en el método onCreate procedemos, mediante la clase RealmConfiguration a darle vida a nuestra base de datos. Es una configuración básica en la que le damos un nombre a nuestra base de datos y le ponemos la versión 1 para el esquema:

<pre><code class="kotlin">class AndroidApplication: Application() {

    override fun onCreate() {
        super.onCreate()
        Realm.init(this)
        Realm.setDefaultConfiguration(RealmConfiguration.Builder()
                .name(DatabaseConstants.NAME)
                .schemaVersion(DatabaseConstants.FIRST_VERSION)
                .build())
    }

}
</code></pre>

Llegados a este punto, vamos a crear nuestra primera entidad de la base de datos, en este caso una Persona con número de identificación, nombre y apellidos. Para ello tenemos que crearnos la clase extendiendo a *RealmObject*, que necesita por defecto un constructor vacío. En *Kotlin* pues hemos de darle valores por defecto a las properties de la clase:

<pre><code class="kotlin">open class Person(@PrimaryKey var identityId: String = "", 
					var name: String= "", 
					var surname: String = "") : RealmObject()
</code></pre>

Ahora si, ya podemos crearnos una persona, guardarla en nuestra base de datos, recuperarla de la misma y por supuesto mostrarla por pantalla en nuestra primera versión de la aplicación.

<pre><code class="kotlin">class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val db = Realm.getDefaultInstance()
        db.beginTransaction()
        db.copyToRealmOrUpdate(Person("00000001R", "Javi", "Rodríguez"))
        db.commitTransaction()
        val firstPerson = db.where(Person::class.java).findFirst()
        label.text = firstPerson?.let {
            it.surname + ", " + it.name + " - " + it.identityId
        }
    }

}
</code></pre>

Podéis ver el ejemplo hasta aquí: [First version](https://github.com/theyavikteam/post_examples/tree/post/20180606-realmmigration/first_version)


## 2 - Vaya, se me ha olvidado la edad

¿Que ocurre si queremos añadir la edad a las personas y ya tenemos nuestra aplicación a producción? Vamos a verlo.

En primer lugar añadimos la edad al objeto *Person*:
<pre><code class="kotlin">open class Person(@PrimaryKey var identityId: String = "", 
					var name: String= "", 
					var surname: String = "", 
					var age: Int = 0) : RealmObject()
</code></pre>

Actualizamos la edad a la persona y vamos a mostrar por pantalla y ejecutamos nuestra aplicación...:
<pre><code class="kotlin">val db = Realm.getDefaultInstance()
        db.beginTransaction()
        db.copyToRealmOrUpdate(Person("00000001R", "Javi", "Rodríguez", 28))
        db.commitTransaction()
        val firstPerson = db.where(Person::class.java).findFirst()
        label.text = firstPerson?.let {
            it.surname + ", " + it.name + ": " + it.age + " - " + it.identityId
        }
</code></pre>

Puuuuummm!!!!! Instacrashhhh!!!!! ¿Que ha pasado? ¿Que me dice la consola?

![2018-6-6_01](https://raw.githubusercontent.com/theyavikteam/theyavikteam.github.io/master/images/2018-6-6_01.png)

Básicamente hemos cambiado el esquema de nuestra base de datos añadiendo la edad a la clase *Person*. El log es tan chivato que me dice como arreglar semejante destrozo. **Una migración**. ¡Vamos a ello!:

Creamos una clase que implemente la interfaz *RealmMigration* y sobreescribimos el método *migrate*. Aquí 

<pre><code class="kotlin">class MyMigration : RealmMigration {
    override fun migrate(realm: DynamicRealm, oldVersion: Long, newVersion: Long) {
        if (DatabaseConstants.FIRST_VERSION == oldVersion) {
            realm.schema.get("Person")?.addField("age", Int::class.java)
            oldVersion.inc()
        }
    }
}
</code></pre>

Y ahora vamos a modificar la configuración de *Realm* para hacer uso de ella:

<pre><code class="kotlin">Realm.setDefaultConfiguration(RealmConfiguration.Builder()
                .name(DatabaseConstants.NAME)
                .schemaVersion(DatabaseConstants.SECOND_VERSION)
                .migration(MyMigration())
                .build())
</code></pre>

¡Y voilá! Ya tenemos nuestra aplicación corriendo de nuevo.

Podéis ver el ejemplo hasta aquí: [Second version](https://github.com/theyavikteam/post_examples/tree/post/20180606-realmmigration/second_version)

## 3 - Además de personas, los perros tienen derecho a existir

Y cómo los perros también tiene derecho a existir en nuestra base de datos vamos a definir una muy sencilla clase para ellos:

<pre><code class="kotlin">open class Dog(@PrimaryKey var name: String= "") : RealmObject()
</code></pre>

Damos de alta un perro en la base de datos, consultamos por él y lo intentamos desplegar en la pantalla de nuestro dispositivo...:
<pre><code class="kotlin">db.beginTransaction()
db.copyToRealmOrUpdate(Dog("Chiki"))
db.commitTransaction()
val firstDog = db.where(Dog::class.java).findFirst()
label.text = firstDog?.name)
</code></pre>

Puuuuummm!!!!! Instacrashhhh!!!!! ¿Que ha pasado? ¿Que me dice la consola?

![2018-6-6_02](https://raw.githubusercontent.com/theyavikteam/theyavikteam.github.io/master/images/2018-6-6_02.png)

Vaya, no me había dado cuenta que hemos añadido una tabla más en nuestra base de datos. Necesito ampliar la migración para contemplar este caso y cambiar la versión al esquema en la configuración de la base de datos:

<pre><code class="kotlin">class MyMigration: RealmMigration{
    override fun migrate(realm: DynamicRealm, oldVersion: Long, newVersion: Long) {
        val realmSchema = realm.schema
        if (DatabaseConstants.FIRST_VERSION == oldVersion) {
            realmSchema.get("Person")?.addField("age", Int::class.java)
            oldVersion.inc()
        }
        if (DatabaseConstants.SECOND_VERSION == oldVersion){
            realmSchema.create("Dog").addField("name", String::class.java, FieldAttribute.PRIMARY_KEY, FieldAttribute.REQUIRED)
            oldVersion.inc()
        }
    }
}
</code></pre>

Si volvemos a ejecutar nuestra aplicación ya no tendremos ningún tipo de problema.

Podéis ver el ejemplo hasta aquí: [Third version](https://github.com/theyavikteam/post_examples/tree/post/20180606-realmmigration/third_version)

## 4 - ¿Y si las personas tienen un perro de mascota?

## 5 - Conclusión


