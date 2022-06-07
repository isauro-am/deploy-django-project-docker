# lldeploy-django-project-docker

To deploy a django project in a docker container exposing the django project folders to be able to edit the project in real time.

This project will have a database under the same network, it can also be configured to an external / remote database

### First we will create our Django project

```
# If you don't have django installed, you will have to install the service.
sudo apt install python3-django

# We will create the project in the root folder of this project (at the level of the docker folder)
django-admin startproject my-project
```

### Once our project is created we will have something similar to the next

```
docker/
	docker-compose.yml
	DockerfilePanel
	pip_requirements/
			base.txt
			test.txt

my-proyect/
	...
```

Now we are going to create inside docker/ the file that contains the environment variables, these variables will be defined according to the needs of the project, the file will be called **.env**

It is very important to define the variable **PROJECT_NAME** as we named our project in the previous step, in this example **my-project**

```
# docker/.env

DATABASE_CONTAINER_NAME=database_container
DATABASE_HOST=my_database_host_name

DATABASE_ROOT_PASSWORD=Secur3PasswDdRoo7
DATABASE_NAME=databaseName
DATABASE_USER=databaseUser
DATABASE_PASSWORD=Secur3PasswDdUseEr

PROJECT_NAME=my-proyect
WEB_PANEL_HOSTNAME=my_project_web_panel
WEB_PANEL_CONTAINER_NAME=web_panel_container

SUB_NETWORK=100.25.0.0/24
DATABASE_IP=100.25.0.10
WEB_PANEL_IP=100.25.0.11

DEBUG=True
LOCAL_HOST_URL=localhost
PUBLIC_HOST_URL=site.name.com
```

Now before we can run our containers, we need to make a change inside our django project

Following our example we will have to navigate to **my-project/my-project/** inside we will create a folder called **settings** , and the **settings.py** file we will move inside the new folder and we will name it as **base.py**

Now inside this folder **settings** we are going to create a new file called **develop.py** inside this file we will put the next content

```python
from .base import *
import environ

env = environ.Env()
environ.Env.read_env()

ALLOWED_HOSTS = [
    '127.0.0.1',
    env('LOCAL_HOST_URL', default=None),
    env('PUBLIC_HOST_URL', default=None),
    ]

DEBUG = env('DEBUG', default=True)

INSTALLED_APPS += (
    'debug_toolbar',
)

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': env('DATABASE_NAME', default=''),
        'USER': env('DATABASE_USER', default=''),
        'PASSWORD': env('DATABASE_PASSWORD', default=''),
        'HOST': env('DATABASE_HOST', default=''),
        # 'PORT': '',
    }
}

MIDDLEWARE += (
    'debug_toolbar.middleware.DebugToolbarMiddleware',
)

TIME_ZONE = 'America/Mexico_City'
```

Basically we are indicating to Django the database to use and its connection data.

Once this is done, inside the docker folder we must execute

```
docker-compose build && docker-compose up -d
```

Now we will have our project in an "isolated" environment and ready to work.

### **Note:**

If you will work a django project inside docker for a project inside Github, Gitlab, etc. You just have to make a copy of the docker folder and move it to the root of your project/repository, then it is highly recommended to create a file in the root of the project called **.gitignore** and add the **.env** extension to it

```
# .gitignore
.env
```

With this we will prevent uploading the real values/passwords of your project to the repository.

---

If your project requires another library to be used by django, you can add it to the list in the file **docker/pip_requirements/base.txt**
