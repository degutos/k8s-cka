# Container image

## Manual steps to build your container 

1. Install OS - Ubuntu 
2. Update apt repo
3. Install dependencies using apt
4. Install python dependencies using pip
5. Copy source code to /opt folder
6. Run web server using flask command 


## Create your own Dockerfile

```
From ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql 

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

Note: Each instructions line above will generate a new layer in the container, it is important to keep those instructions consistent and short in order to create a smaller image.

## Build your container

```
$ docker build Dockerfile -t degutos/my-custom-app
```

### Send our new image to docker hub repository

```
$ docker push degutos/my-custom-app
```

### Check container layers

Once we created our new image we can observe the container layers with the following command:

```
$ docker history degutos/my-custom-app
```

Each line in the Dockerfile will represent a new layer in the container.
If you need to rebuild your image either because it failed or you need update your application source code the docker image command will re-build only those layers needed, this will optimize your time build and space.



## Another Dockerfile sample:

```
$ cat Dockerfile 
FROM python:3.6

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```

## Building our container image

```
$ docker build -t webapp-color . 
Sending build context to Docker daemon  125.4kB
Step 1/6 : FROM python:3.6
 ---> 54260638d07c
Step 2/6 : RUN pip install flask
 ---> Running in 9b4d191fc7be
Collecting flask
  Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
Collecting click>=7.1.2
  Downloading click-8.0.4-py3-none-any.whl (97 kB)
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting Jinja2>=3.0
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Collecting importlib-metadata
  Downloading importlib_metadata-4.8.3-py3-none-any.whl (17 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.0.1-cp36-cp36m-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (30 kB)
Collecting dataclasses
  Downloading dataclasses-0.8-py3-none-any.whl (19 kB)
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.1.1-py3-none-any.whl (26 kB)
Collecting zipp>=0.5
  Downloading zipp-3.6.0-py3-none-any.whl (5.3 kB)
Installing collected packages: zipp, typing-extensions, MarkupSafe, importlib-metadata, dataclasses, Werkzeug, Jinja2, itsdangerous, click, flask
Successfully installed Jinja2-3.0.3 MarkupSafe-2.0.1 Werkzeug-2.0.3 click-8.0.4 dataclasses-0.8 flask-2.0.3 importlib-metadata-4.8.3 itsdangerous-2.0.1 typing-extensions-4.1.1 zipp-3.6.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 21.2.4; however, version 21.3.1 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
Removing intermediate container 9b4d191fc7be
 ---> a4cdc309360d
Step 3/6 : COPY . /opt/
 ---> 7c9d940f0e55
Step 4/6 : EXPOSE 8080
 ---> Running in 9bd491380cd8
Removing intermediate container 9bd491380cd8
 ---> 6b4f5a079373
Step 5/6 : WORKDIR /opt
 ---> Running in ee4799897b85
Removing intermediate container ee4799897b85
 ---> 8ac166256733
Step 6/6 : ENTRYPOINT ["python", "app.py"]
 ---> Running in 605f60ae27f5
Removing intermediate container 605f60ae27f5
 ---> cc4324a3de20
Successfully built cc4324a3de20
Successfully tagged webapp-color:latest



$ docker images
REPOSITORY                      TAG           IMAGE ID       CREATED          SIZE
webapp-color                    latest        cc4324a3de20   44 seconds ago   913MB
mysql                           latest        7ce93a845a8a   4 weeks ago      586MB
alpine                          latest        324bc02ae123   4 weeks ago      7.79MB
nginx                           latest        a72860cb95fd   2 months ago     188MB
nginx                           alpine        1ae23480369f   2 months ago     43.2MB
ubuntu                          latest        35a88802559d   2 months ago     78MB
redis                           latest        6c00f344e3ef   3 months ago     116MB
postgres                        latest        07a4ee949b9e   3 months ago     432MB
python                          3.6           54260638d07c   2 years ago      902MB
nginx                           1.14-alpine   8a2fb25a19f5   5 years ago      16MB
kodekloud/simple-webapp-mysql   latest        129dd9f67367   5 years ago      96.6MB
kodekloud/simple-webapp         latest        c6e3cd9aae36   5 years ago      84.8MB
$ 
```

## Running a container on port 8080 on the container and port 8282 on the host from a image

```
$ docker run -p 8282:8080 webapp-color
 This is a sample web application that displays a colored background. 
 A color can be specified in two ways. 

 1. As a command line argument with --color as the argument. Accepts one of red,green,blue,blue2,pink,darkblue 
 2. As an Environment variable APP_COLOR. Accepts one of red,green,blue,blue2,pink,darkblue 
 3. If none of the above then a random color is picked from the above list. 
 Note: Command line argument precedes over environment variable.


No command line argument or environment variable. Picking a Random Color =darkblue
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.12.0.2:8080/ (Press CTRL+C to quit)
 ```


