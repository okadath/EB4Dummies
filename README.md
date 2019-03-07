# Tutorial deploy EB 
Manejar todo en un virtualenv
```
virtualenv  venv(usa por default python2.7)
source venv/bin/activate
```
Para cerrarlo usar `source deactivate`

### Configurando el proyecto

Configurar un proyecto de demostracion (si ya tienes proyecto salta a la siguiente seccion)
```
git clone https://github.com/realpython/image-of-the-day.git
cd image-of-the-day/
git checkout tags/start_here_py3
pip install -r requirements.txt
```

Usaremos una base de datos de postgres
si tienes problemas ejecutandolo en local hay que crear la tabla, logeado en Postgres usamos `\l` para listar tablas y `\q` para salir de la consola
![Screenshot postgres](https://raw.githubusercontent.com/okadath/EB4Dummies/master/postgres.png)

<!-- para subirlo a HTML debe de ser solo el nombre del archivo!!!! -->
```
sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'pass';"
sudo service postgresql restart
sudo -u postgres psql
create database iotd with owner postgres;
```
Instalar la libreria de python psycopy2 para conectarse a Postrgres
```
pip install psycopy2
```
En AWS ya viene preconfigurado entonces este problema no ocurre en remoto
este proyecto necesita configuraciones iniciales:
```
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

En `iotd/management/commands/createsu.py` esta el usuario por default

Este proyecto al solo poseer una consola de Admin solo despliega el manager en  `http://localhost:8000/admin`

### Consola EB
Instalar el cliente de AWS para python:
```
pip install awsebcli
eb --version
```

inicializar el proyecto
 ```
 eb init
 ```
y configurar el proyecto, al momento de solicitar las credenciales debes haber creado tu ssh key en la seccion "My security credentials" creas la clave y la guardas (Si la pierdes puedes perder el acceso al proyecto entero!)

![Seccion My security credentials](https://raw.githubusercontent.com/okadath/EB4Dummies/master/keys.png)







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