# TP final para la materia Administración de sistemas GNU/Linux y Virtualización 2020
TP final realizado junto a Federico Di Nardo, el cual consiste en crear una página web con Wordpress con Docker.

## Cómo utilizar estos archivos


### 1ro: Descarga a una carpeta
Se encontrarán los archivos *docker-compose.yml* y *php.ini*. El primero es el código fuente para ejecutar Docker y el segundo es una configuración para PHP de ejemplo.


### 2do: Explicación

Podemos ver que se divide en tres partes principales, donde cada una de las partes va a ser un contenedor distinto. Tenemos al contenedor de la base de datos, al contenedor propio del wordpress y a un contenedor para administrar la base de datos.

**Contenido del docker-compose.yml:**
```
version: '3.3'

services:
        db:
                container_name: mysql-06
                env_file: .env
                image: mysql:${MYSQL_TAG}
                volumes:
                        - ./db_data:/var/lib/mysql
                environment:
                        MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
                        MYSQL_DATABASE: ${MYSQL_DATABASE}
                        MYSQL_USER: ${MYSQL_USER}
                        MYSQL_PASSWORD: ${MYSQL_PASSWORD}

        wordpress:
                container_name: wp-06
                depends_on:
                        - db
               
                image: wordpress:latest
                volumes:
                        - ./src/themes:/var/www/html/wp-content/themes
                        - ./src/plugins:/var/www/html/wp-content/plugins
                        - ./src/uploads:/var/www/html/wp-content/uploads
                        - ./php.ini:/usr/local/etc/php/conf.d/php.ini
                ports:
                        - "8000:80"
                env_file: .env
                restart: ${RESTART}

                environment:
                        WORDPRESS_DB_HOST: db:3306
                        WORDPRESS_DB_USER: ${MYSQL_USER}
                        WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
                        WORDPRESS_DB_NAME: ${MYSQL_DATABASE}


        phpmyadmin:
                container_name: php-06
                image: phpmyadmin/phpmyadmin
                depends_on:
                        - db
                environment:
                        MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
                ports:
                        - "8080:80"
                restart: ${RESTART}
                       
volumes:
        db_data: {}
```


#### Container de mySQL

Como se ve en el container de mySQL, definimos el *env_file*, en donde vamos a guardar los valores de nuestras **variables de entorno**. Justamente las variables van a estar en el fichero *.env*. las cuales son: 
+ MYSQL_ROOT_PASSWORD: esta variable es obligatoria y lo que hace es especificar la contraseña que se establecerá para la cuenta root de usuario de MySQL.
+ MYSQL_DATABASE:  Esta variable es opcional y lo que hace es especificar el nombre de una base de datos que se creará al iniciar la imagen.
+ MYSQL_USER:  
+ MYSQL_PASSWORD: estas variables son opcionales, se utilizan en conjunto para crear un nuevo usuario y establecer la contraseña de ese usuario. 

Usamos variables de entorno, porque este contenedor ya tiene en su script de inicio leer las variables de entorno y generar el archivo de configuración basado en estas variables de entorno. De esta forma yo puedo usar la misma imagen sin tener que modificar los archivos de adentro

Además especificamos un **volumen**, para que los datos queden guardados y no se pierdan. Si no especificamos el volumen, los datos que tenga en este contenedor no se van a guardar. Entonces, cada vez que levanto este contenedor, se leen los datos de ese volumen.


#### Container de Wordpress

Este container va a depender de la base de datos. Entonces le vamos a indicar que tenga **dependencia** con la base de datos mediante el comando: *depends_on: db*

Algo muy importante son los **volúmenes**. En ellos se van a indicar las carpetas que hacen referencia a nuestro disco duro. Vamos a indicarle que la raíz, o sea en *src*, va a estar la carpeta *plugins*. Ahí le indicamos la carpeta hacia adentro del contenedor. Entonces, vamos a indicarle que la instalación de ese wordpress está en *var/www/html/wp-conten/plugins* (si la carpeta no existe, la crea). Entonces los plugins se van a descargar, o van a hacer referencia a la ruta dentro de ese servidor, a esta carpeta (plugins). Así igual con los temas y uploads.

Wordpress funciona en los **puertos** 80 y 443. Es muy común que estos puertos estén ocupados, por lo que vamos a indicarle que utilice otro puerto. En nuestro caso, le vamos a indicar que utilice el puerto 8000.

Para que la instancia se pueda crear correctamente, vamos a declarar las siguientes **variables de entorno**. Esto va a ser muy parecido a lo que hicimos con el contenedor de la base de datos. Vamos a declarar las instancias de: 
+ WORDPRESS_DB_HOST: acá le vamos a indicar el nombre del contenedor, y le indicamos el puerto en el cual funciona el mysql
+ WORDPRESS_DB_USER:
+ WORDPRESS_DB_PASSWORD: obviamente estos datos del password tienen  que coincidir con la base de datos
+ WORDPRESS_DB_NAME: aca le ponemos el nombre de la base de datos


#### Container de phpMyAdmin

PhpMyAdmin es una herramienta que permite administrar bases de datos MySQL. Wordpress utiliza una base de datos MySQL para guardar todo tipo de información de las páginas webs. La base de datos almacena cada opción de Wordpress, temas, plugins, usuarios, comentarios y todo tipo de información del sistema que esté almacenado dentro de ella. Mediante esta herramienta podemos crear, eliminar y editar tablas y campos, optimizar bases de datos, ejecutar sentencias SQL, exportar o importar la base de datos, crear una copia de seguridad, etc. Las bases de datos contienen la información estructurada de toda nuestra página web, y cada vez que se carga una página web se consulta a la base de datos para cargar las páginas, las imágenes, los textos, etc.

Como va a depender de la base de datos, por lo cual en **dependencias** le ponemos *depends_on: db*.

En la variable de entorno, *MYSQL_ROOT_PASSWORD* es una variable obligatoria, y justamente especifica la contraseña que se establecerá para la cuenta root de superusuario.


#### Environment

Lo que hicimos fue hacer un script aparte del docker-compose, en donde se van a guardar los valores de cada una de las variables de entorno que nosotros querramos. Esto se suele utilizar así, ya que si queremos en algún momento modificar alguna, es mucho más sencillo y seguro modificar el script que tenemos con las variables que tener que modificar el docker-compose. Además de que cada uno puede definir sus propias contraseñas para trabajar en un entorno local sin depender de otra configuración.
En el docker-compose, se utiliza la interpolación de cadena (lo que llamamos a la notación ${} ) para asignar los valores de nuestras variables *.env* a las variables de entorno. **Este archivo debe crearse en la misma carpeta donde se descargaron los otros archivos.**

**Ejemplo de archivo *.env*:**
```
MYSQL_ROOT_PASSWORD=xxxxxxxx
MYSQL_DATABASE=xxxxxxxx
MYSQL_USER=xxxxxxxx
MYSQL_PASSWORD=xxxxxxx
MYSQL_TAG=xxxxxxxx
RESTART=always
```

#### PHP.ini

Existen algunas configuraciones recomendadas para php.ini que hacen que Wordpress funcione mejor. 

+ Limitar la memoria: Como podemos observar por su nombre, *memory_limit* es el comando para limitar el uso de memoria PHP en wordpress. El valor que se define debe ser mayor que el archivo que estamos intentando cargar. Esta es la memoria necesaria para cargar archivos y ejecutar comandos. 

+ Max_execution_time: Este comando define el tiempo necesario para ejecutar cada script. En otras palabras, define el tiempo que el servidor necesita para    ejecutar los comandos script. Por ejemplo, si estamos cargando un script muy grande en el servidor, tomará más de unos segundos. Por lo tanto, debemos eliminar el tiempo de ejecución o aumentarlo.

+ Post_max_size: Este comando define los datos máximos que pueden tener  una publicación. Para eliminar la limitación, cambiamos el valor a 0. Al usar método POST podemos llamar a la publicación desde el servidor.

+ Upload_max_filesize: Este comando define el archivo máximo que podemos cargar en Wordpress. Podemos ver el número máximo de carga en la galería de Wordpress. Este comando es quien define esa limitación. Si tenemos un problema del error de agotamiento del límite de memoria, este comando debe ser eliminado o aumentado.

+ Max_input_time: Este comando define los tiempos necesarios para que cada dato sea analizado. Datos como POST y GET. El tiempo comienza exactamente cuando el comando hace la solicitud al PHP del servidor y termina cuando se inicia el comando. El valor predeterminado es -1; para eliminar esta limitación, se debe colocar 0.


### 3do: Cómo correrlo

Para poder ejecutar el *docker-compose.yml* utilizar los siguientes comandos:
**La primera vez**
`sudo docker-compose build`
**Luego**
`sudo docker-compose up`

Para ir a la página web creada, debemos ir al brower y escribir en la barra de direcciones: *localhost:8080*



### 4to: Cómo usar PLUGINS y THEMES

Vamos a detallar el paso a paso de la instalación de los Plugins. Primero, vamos a ir a la página wordpress.org. Ahí vamos a ir al ítem Plugins, en donde van a aparecer todos los plugins que podemos descargar. Aquellos que tienen una etiqueta de PRO, es que son pagos, el resto van a ser gratis. Para nuestra página, vamos a descargar el plugin de **ELEMENTOR**, por lo que estuvimos viendo es uno de los más comunes y muy simple de usar.  Una vez descargado, se va a guardar en la carpeta Downloads. Entonces, lo movemos a la carpeta *plugins* que habíamos creado antes. Una vez ahí, pasamos a descomprimirlo, ya que está comprimido como un archivo ZIP. 
Así de sencillo, con estos simples pasos podemos cargar cualquier plugins a nuestro docker-compose.

La instalación de los temas se realiza de igual manera que los plugins. Vamos a ir a la misma página wordpress.org , y ahí vamos a seleccionar temas. Nosotros descargamos el **ASTRA**, el cual es uno muy usado. Los pasos para la descompresión son los mismos, excepto que ahora lo vamos a mover a la carpera *src/themes*.
Una vez descomprimido en esa carpeta, cuando iniciemos nuestro docker-compose ya va a estar listo para su uso. 
De esta forma, podemos cargar los plugins y temas que queramos.



