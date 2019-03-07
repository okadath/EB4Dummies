# Tutorial deploy EB 
### Configurando el proyecto

Configurar un proyecto de demostracion (ya deberia haber uno, si no solo seguir al pie de la letra)
```
git clone https://github.com/realpython/image-of-the-day.git
cd image-of-the-day/
git checkout tags/start_here_py3
virtualenv  (usa por default python2.7)
source venv/bin/activate
pip install -r requirements.txt
```

usaremos una base de datos de postgres
si tienes problemas ejecutandolo en local hay que crear la tabla, logeado en postgres usamos `\l` para lsitar tablas y `\q` para salir de la consola:

```
sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'pass';"
sudo service postgresql restart
sudo -u postgres psql
create database iotd with owner postgres;
```

en AWS ya viene preconfigurado entonces este problema no ocurre en remoto
este proyecto necesita configuraciones iniciales:
```
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```
en `iotd/management/commands/createsu.py` esta el usuario por default

### Consola EB







