# Tutorial deploy EB 
Manejar todo en un virtualenv
```bash
virtualenv  venv(usa por default python2.7)
source venv/bin/activate
```
Para cerrarlo usar `source deactivate`

### Configurando el proyecto

Configurar un proyecto de demostracion (si ya tienes proyecto salta a la siguiente seccion)
```bash
git clone https://github.com/realpython/image-of-the-day.git
cd image-of-the-day/
git checkout tags/start_here_py3
pip install -r requirements.txt
```

Usaremos una base de datos de postgres
si tienes problemas ejecutandolo en local hay que crear la tabla, logeado en Postgres usamos `\l` para listar tablas y `\q` para salir de la consola
![Screenshot postgres](https://raw.githubusercontent.com/okadath/EB4Dummies/master/postgres.png)

<!-- para subirlo a HTML debe de ser solo el nombre del archivo!!!! -->
```bash
sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'pass';"
sudo service postgresql restart
sudo -u postgres psql
create database iotd with owner postgres;
```
Instalar la libreria de python psycopy2 para conectarse a Postrgres
```bash
pip install psycopy2
```
En AWS ya viene preconfigurado entonces este problema no ocurre en remoto
este proyecto necesita configuraciones iniciales:
```bash
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

En `iotd/management/commands/createsu.py` esta el usuario por default

Este proyecto al solo poseer una consola de Admin solo despliega el manager en  `http://localhost:8000/admin`


Comandos de superusuario:
```bash
python manage.py createsuperuser
python manage.py changepassword <user_name>
```

Para pasar un usuario normal a admin:

`python manage.py shell`

```python
from django.contrib.auth.models import User
user = User.objects.get(username='normaluser')
user.is_superuser = True
user.save()
```

### Consola EB
Instalar el cliente de AWS para python:
```bash
pip install awsebcli
eb --version
```

inicializar el proyecto con:
 ```
 eb init
 ```
y configurarlo, al momento de solicitar las credenciales debes haber creado tu ssh key en la seccion "My security credentials" creas la clave y la guardas (Si la pierdes puedes perder el acceso al proyecto entero!)

![Seccion My security credentials](https://raw.githubusercontent.com/okadath/EB4Dummies/master/keys.png)



Una vez con las claves ya hechas configuramos con `eb init`

En *Select a default region* elegir alguna segun zona geografica (solo oprimes enter)

![init instance](https://raw.githubusercontent.com/okadath/EB4Dummies/master/ebinit1.png)

Al ingresar la aplicacion 
```
Select an application to use
1) image-of-the-day
2) [ Create new Application ]
```
elegir la deseada (aqui 2) y elegir su nombre

Si usas Docker Para crear la instancia te lo preuntara, escribes Yes(Y)

Al usar CodeCommit eliges No(N), usamos Github

En SSH elegimos Yes(Y) y elegimos la llave creada anteriormente

![init](https://raw.githubusercontent.com/okadath/EB4Dummies/master/ebinit4.png)


Despues de esto al escribir `eb console` nos llevara a AWS en la instancia creada

![console aws](https://raw.githubusercontent.com/okadath/EB4Dummies/master/ebconsole.png)


### Deployando cambios 
Despues usar `eb create` para subir el proyecto en formato zip(es muy lento, use codenvy para que fuese rapido) usamos la configuracion por default dandole enter a todo
![create project](https://raw.githubusercontent.com/okadath/EB4Dummies/master/ebcreate.png)


Si se hacen cambios en el proyecto se debe de usar git y commitearlos para despues usar 
`eb deploy` para actualizar el server

A partir de aqui son configuraciones de la instancia y del servidor
Para usar el script wsgi usamos `eb config` y eso abrira el editor en consola por defecto (instalar nano!!)

![eb config](https://raw.githubusercontent.com/okadath/EB4Dummies/master/ebconfig.png)

Ahi buscaremos la siguiente linea y la cambiaremos para usar el`wsgi.py` de la aplicacion
 ```yaml
 aws:elasticbeanstalk:container:python:
    NumProcesses: '1'
    NumThreads: '15'
    StaticFiles: /static/=static/
    WSGIPath: vitalquestions/wsgi.py
  aws:elasticbeanstalk:container:python:staticfiles:
    /static/: static/
  ```
  ![change config](https://raw.githubusercontent.com/okadath/EB4Dummies/master/ebconfig2.png)

Despues de eso crearemos algunos archivos en `.ebextensions` para ejecutar automaticamente scripts en EC2 para configurar nuestro servidor:
+ 02_python.config:

```yaml
container_commands:
  01_migrate:
    command: "source /opt/python/run/venv/bin/activate && python manage.py migrate --noinput"
    leader_only: true
```
+ 03_apache.config:
```yaml
container_commands:
  01_setup_apache:
    command: "cp .ebextensions/enable_mod_deflate.conf /etc/httpd/conf.d/enable_mod_deflate.conf"
```
+ enable_mod_deflate.conf

```xml
# mod_deflate configuration 
<IfModule mod_deflate.c> 
  # Restrict compression to these MIME types 
  AddOutputFilterByType DEFLATE text/plain 
  AddOutputFilterByType DEFLATE text/html 
  AddOutputFilterByType DEFLATE application/xhtml+xml 
  AddOutputFilterByType DEFLATE text/xml 
  AddOutputFilterByType DEFLATE application/xml 
  AddOutputFilterByType DEFLATE application/xml+rss 
  AddOutputFilterByType DEFLATE application/x-javascript 
  AddOutputFilterByType DEFLATE text/javascript 
  AddOutputFilterByType DEFLATE text/css 
  # Level of compression (Highest 9 - Lowest 1) 
  DeflateCompressionLevel 9 
  # Netscape 4.x has some problems. 
  BrowserMatch ^Mozilla/4 gzip-only-text/html 
  # Netscape 4.06-4.08 have some more problems 
  BrowserMatch ^Mozilla/4\.0[678] no-gzip 
  # MSIE masquerades as Netscape, but it is fine 
  BrowserMatch \bMSI[E] !no-gzip !gzip-only-text/html 
<IfModule mod_headers.c> 
  # Make sure proxies don't deliver the wrong content 
  Header append Vary User-Agent env=!dont-vary 
</IfModule> 
</IfModule>

```

Hay un error al hacer el deploy ya habiendo configurado la instancia:

![hosts](https://raw.githubusercontent.com/okadath/EB4Dummies/master/allowedhost.png)

Agregar a `ALLOWED_HOST` el nombre del server en `<nombre_aplicacion>/settings.py`

```python
ALLOWED_HOSTS = ["20vq-codenvy-dev.us-west-2.elasticbeanstalk.com",]
```


### Corriendo el server

Despues de todo esto el dashboard deberia lucir asi, el mensaje de Healt indica el estado en el que se haya el proyecto, ya [corriendo](#todo) en AWS.

![dashboard](https://raw.githubusercontent.com/okadath/EB4Dummies/master/funcionando.png)

El status indica que es severo por que el servidor trata de verificar las conexiones como si abriera la pagina normalmente a la URL (nosotros la tenemos en URL/admin) asi que al no poder conectarse nos muestra un error del tipo 5XX(error en la conexion) lo cual se arregla simplemente indicando una pagina principal apuntando a nuestra URL:

![errores en el monitor](https://raw.githubusercontent.com/okadath/EB4Dummies/master/severe.png)


El servidor ya se encuentra funcionando despues de todo esto, se accede a el con la URL del proyecto en la seccion "direccion_del_proyecto/admin".

![URL del proyecto](https://raw.githubusercontent.com/okadath/EB4Dummies/master/URL.png)

Este proyecto solo posee un administrador de usuarios, el cual ya se encuentra corriendo en Internet:

![login](https://raw.githubusercontent.com/okadath/EB4Dummies/master/login.png)



### TODO 
+ Hay que buscar una forma segura de apagar el server sin perdida de datos ya que al estar siempre encendido se cobra por su uso y este solo es un demo

Se puede acceder a lso costos de las instancias dando click en la siguiente pestaña y ahi nos indicara el monto cargado por la instancia

![billing](https://raw.githubusercontent.com/okadath/EB4Dummies/master/billingmenu.png)

![pricing](https://raw.githubusercontent.com/okadath/EB4Dummies/master/pricing.png)

si hay errores con los js es por que falto correr el collectstatic para condensar los js y los css en la carpeta `/static/`
hay que agregar al settings :
```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [os.path.join(BASE_DIR, "static")]

# media files
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'


PROJECT_DIR = os.path.dirname(os.path.abspath(__file__))
STATIC_ROOT = os.path.join(PROJECT_DIR, 'static')
```
hay que asegurarse que si esten los js en la carpeta, si no pasarlos a mano Ctrl+C -> Ctrl+V

```python
 python manage.py collectstatic
```
