# - Dockerised App - 

## - Summary -
#### In this project, I will build a simple Python web app, put it into a Docker image, and then run it as a container. I will then push the image to Docker Hub.

## - Set Up -
#### To begin this project, I completed the following steps:

- Python 3
- Docker Desktop
- Created a Docker Hub account

## - Creating a simple Python web app -
#### I decided to use Flask for my simple web app. I created a file called app.py and inserted the following code: 

```
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from my Dockerised Python app!"

@app.route("/about")
def about():
    return "This app is running inside a Docker container."

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5001)
```

## - Creating a requirements file -
#### The reason for creating this file is very important, as it will list all of the Python packages that my app needs. In my case, the app depends on Flask version 3.1.0 . The beauty of having a requirements file is that if anyone else were to use my Docker image, they would need to run the following command to install everything automatically: 

```
python3 -m pip install -r requirements.txt
```

## - Testing Locally - 
#### Before I start using my app within a Docker container, I would like to test it locally to ensure it works as intended. To do so, I will run the following commands: 

```
python3 -m install -r requirements.txt
python3 app.py
```
![app](images/appPY.png)

#### When I will then go to the following web address to see if my web app has launched successfully: "http://localhost:5001". If it was successful, then I should see my welcome message.

![welcomeimage](images/welcome.png)

## - Creating the Dockerfile - 
#### A Dockerfile will define how the image is built: base image, working directory, copied files, installed dependencies, exposed port, and startup command. Docker’s Dockerfile overview uses this same structure for Python. In my Dockerfile, I have the following:

```
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5001

CMD ["python", "app.py"]
```

#### I will break down the seven commands included in my Dockerfile:

#### FROM python:3.12-slim
- This command sets the base image from Docker Hub. I'm using the official Python image from Docker Hub. I've set the version to 3.12 and I'm using the 'slim' version. This means that Python does not need to be installed manually. It also ensures consistency as everyone who uses this image will be using the same Python version.

#### WORKDIR 
- This command will set my working directory inside of the container, this will ensure that all future commands will run inside /app . This will keep my project organised and will avoid messy file paths. If I did not do this then all files would go to 'root/' which does not follow best practice.


#### COPY
- The copy command will copy 'requirements.txt' from my Mac into the container; the '.' means that the file will be copied into my current directory, which will be /app, which was set with the WORKDIR command earlier. Because of Docker's layer caching, if requirements.txt does not change, then Docker will skip reinstalling dependencies; this will make it build much faster.

#### RUN
- The RUN command will install the Python dependencies inside of the image; the dependencies are found within requirements.txt, which in this case is a specified version of Flask. I have also set up the image to not store 'pip cache' as this will keep the image smaller. Without the RUN command, then Flask would not exist within the container, which would cause a fatal error.

#### COPY . .
- A second COPY command is inserted as this will copy everything from the project folder into the /app container; the reason I have done this after the pip install command is that if any code change was made, then it would force a reinstall of the dependencies, which is slow and not optimised. 

#### Expose
- This command will document that the container will be using port 5001; it will not actually open port 5001, just its metadata. The actual port mapping will occur later on with the use of a different command.

#### CMD
- This command defines the default command for when the container starts; I have used 'exec form' as it is more reliable and also handles signals correctly, which is important for containers. 

##  - Docker Workflow -
#### When I run the following command, Docker will do the following

```
docker build -t my-app .
```

-1) Pulls python:3.12-slim
-2) Creates /app
-3) Copies requirements.txt
-4) Installs Flask
-5) Copies the code
-6) Saves everything as an image

#### Once my image has been built, I can use the following command to create my container

```
docker run -p 5001:5001 my-app
```

#### Once this command has been run, it will trigger Docker to do the following

- 1. Start a container
- 2. Run: python app.py
- 3. Start the Flask app
- 4. Port 5001 to connect to my browser

## - Creating a .dockerignore file - 
#### A Docker ignore file is very similar to a git ignore file; the core purpose of one is that it will tell Docker not to copy certain files and folders into the image. Without one, Docker will send everything, which will result in slower builds, larger image sizes, and also potential security risks. One line I want to focus on is the '.env'; this may contain API keys, passwords, and secrets. From a security perspective, it is very important that these do not end up in the image and especially that they are not pushed to Docker Hub.


## - Creating the Docker Image - 
#### Now that I have created my Dockerfile, I can now use it to create my image. I do this by running the following command.

```
docker build -t flaskapp .
```
#### I have used the -t command to tag the name "flaskapp" to the Docker image to follow best practice.

![build](images/dockerBuild.png)

#### As can be seen in the image, the Dockerfile successfully created a Docker image. I can also use the "docker images" command to check this.

![images](images/dockerImages.png)

## - Running the Container -
#### Now that Docker has created the Docker image, I can now use this to create my Docker container. I will do this with the following command.

```
docker run -d -p 5001:5001 --name flask-container flaskapp
```
### -d
- #### I will be using the "-d" command to run the container in detached mode so that my terminal will not be "stuck" showing logs. 

### -p
- #### I will also use the "-p" command to port map my container {HOST_PORT : CONTAINER_PORT}. The reason this is needed is that a browser cannot connect to a container directly, so when someone goes to localhost:5001, Docker will send the traffic into the container. 

### --name
- #### I have also used the "--name" command to give a name to the container instead of it having a random ID number; this will make it easier to reference it with commands in CLI. 

![dockerRun](images/dockerRun.png)

#### After running the command, I can see that it is working by opening my browser and going to the following: https://localhost:5001

![Running](images/dockerRunning.png) 

## - Docker Commands -
#### Now that I have my container up and running, I can use the following commands.

```
docker ps
```
#### This will show the current running containers that I have

![ps](images/dockerps.png) 

#### I can also see the current logs by running this command:
```
docker logs flask-container
```
![logs](images/dockerlogs.png)

#### I can also use stop the container very easily using the 'docker stop' command as I gave my container a name so that I could reference it in my CLI.

```
docker stop flask-container
```
#### Finally, I can remove the container using the rm command:

```
docker rm flask-container
```
![dockerRM](images/dockerrm.png) 

## - Creating a Docker Hub Repository -
#### Creating this will allow me to share my projects with others. Everything up until now has all been performed locally. It is not best practice to run containers from your local machine as if my machine fails, then the availability to the container also goes down. By using Docker Hub, I will be able to store my images online and share them with others.

![dockerHub](images/dockerHubRepo.png)


#### I will now tag my image using the following command. Tagging is important as it helps developers with version control. It also allows us to have multiple versions of images.

```
docker tag flaskapp cptbarnett101/flaskapp:latest
```
![tag](images/dockerTag.png) 

#### Now that I have tagged the image, I need to push it to Docker Hub with the following command: 

```
docker push cptbarnett101/flaskapp:latest
```
![push](images/dockerPush.png)

## - Pulling the image from Docker Hub -
#### To validate that the app works independently of my machine, I will remove my local image to simulate a fresh environment. I will then pull the image from Docker Hub.

#### First, I will stop the container and then I shall remove it:

```
docker stop flask-container
docker rm flask-container
```

#### I will then use the 'rmi' command to remove the image from my machine:

```
docker rmi cptbarnett101/flaskapp:latest
```

![rmi](images/dockerRMI.png)

#### Next, I will use the pull command to retrieve the image stored in Docker Hub.

```
docker pull cptbarnett101/flaskapp:latest
```
#### After running this, the following will occur:

- Connection to Docker Hub is formed.
- The repository is found.
- The image layers will be downloaded.
- The image will be reconstructed locally

![pull](images/dockerPull.png) 

#### I will now run the pulled image to see if it has worked as intended. I will do this with the following command:

```
docker run -d -p 5001:5001 --name test-container cptbarnett101/flaskapp:latest
```
![remoteTest](images/dockerTestPulled.png)

#### As can be seen, my container is running successfully, which means my pulled image has worked successfully. 

## - Stopping the Container -
#### Now that I have finished using my Docker container, I need to shut it down properly. First, I will see which of my containers are running using the "ps" command:

```
docker ps
```

#### I will then run the Docker stop command to shut down the container:

```
docker stop test-container
```

![stop](images/dockerStop.png)

