# Desafio DevOps Pro

![GitHub top language](https://img.shields.io/github/languages/top/hbatistuzzo/desafio_devops_pro)
![GitHub commit activity](https://img.shields.io/github/commit-activity/m/hbatistuzzo/desafio_devops_pro)
![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/hbatistuzzo/desafio_devops_pro)
![GitHub last commit](https://img.shields.io/github/last-commit/hbatistuzzo/desafio_devops_pro)

This git repository centralizes the study of the material provided by Fabricio Veronez for the **DevOps & Cloud Challenge** that happened from Jan 13-17 2025, which concerned the practice of the following technologies:

<br>

<table align="center" style="border-collapse: collapse;">
  <tr>
    <td align="center" style="border: none;">
      <img alt="Docker" height="75" src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/docker/docker-original.svg"/>
      <br>Docker
    </td>
    <td align="center" style="border: none;">
      <img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/kubernetes/kubernetes-original.svg" alt="Kubernetes" height="75"/>
      <br>Kubernetes
    </td>
    <td align="center" style="border: none;">
      <img alt="AWS" height="75" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/amazonwebservices/amazonwebservices-plain-wordmark.svg" />
      <br>AWS
    </td>
    <td align="center" style="border: none;">
      <img alt="GitHub Actions" height="75" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/githubactions/githubactions-original.svg" />
      <br>GitHub Actions
    </td>
    <td align="center" style="border: none;">
      <img alt="Prometheus" height="75" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/prometheus/prometheus-original.svg" />
      <br>Prometheus
    </td>
    <td align="center" style="border: none;">
      <img alt="Grafana" height="75" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/grafana/grafana-original.svg" />
      <br>Grafana
    </td>
    <td align="center" style="border: none;">
      <img alt="Grafana" height="75" src="https://www.vectorlogo.zone/logos/nginx/nginx-icon.svg" />
      <br>NGINX
    </td>
  </tr>
</table>



<br>
        
Namely, the purpose is to provide an e-commerce solution for the continuous delivery of an online store app built on python and containerized through Docker. Their inventory is built as a PostgreSQL database. The application will be run on a Kubernetes cluster and will be monitored through Prometheus and exhibiting the relevant information on Grafana. We'll host the application on an AWS EC2 instance and automate the delivery process through CI/CD pipelines provided by the GitHub Actions platform.

<div align="center">

# $\color{goldenrod}{\textrm{1 - Docker}}$

</div>


DevOps aims to automate tasks and standardize processes involved in the application development lifecycle. This is where containers are helpful, since they allow the execution of processes in isolated fashion, avoiding conflict between them, solving the "_but it works on my machine_" conundrum. It ensures the application runs the same way regardless of the environment or OS. In a nutshell, it enforces isolation, portability, initialization speed and consistency in the app behavior. Docker is by far the most adopted container solution, and the one we'll use here.

Docker architecture is divided in 3 elements:
- **The Daemon**, which manages and handles the control of images, containers, networks and volumes;
- **The Client**, which interacts with the Daemon through CLI, allowing the execution of commands such as `docker run`, `docker pull`, etc; 
- **The Registry**, which is the image repository. DockerHub is the official registry.

We interface with Docker through either **Docker Desktop** (a handy GUI with additional resources and extensions, but heavier, with slower perfomance) or **Docker Engine**. I'm personally using the Engine in Ubuntu 24.04 through WSL2.

The overall usage, commands and functionalities of Docker are covered in another (private, for now) repository, `Docker Study`.

Our e-commerce application webpage will be hosted through **NGINX**: `docker container run -d nginx`; I can access this container by using `docker container exec -it <container_id> /bin/bash` and test it by inserting `curl http://localhost` (notice that we're now inside the container CLI, in interactive mode, thanks to the `it` tag), which displays a message confirming that NGINX is running fine.

But in order to access NGINX properly we should set the port binding, where we will route the 80 port from the container to our host machine local port (8080): `docker container run -d -p 8080:80 nginx`.

Now, the PORTS section for this container will exhibit `0.0.0.0:8080->80/tcp` i.e. we have _published the port 80 of the container to the 8080 port on the host (my machine)_, so any requests sent to http://localhost:8080 on my host are forwarded to port 80 of the NGINX server running inside the container. If we open a browser and access `localhost:8080`, we'll see the "welcome to NGINX" webpage.

To stop, we use `docker container stop <container-id>`; we can now remove this container with `docker container rm <container-id>` if we want to do so. 

>__TIP__ Suppose you have a bunch of containers running. `docker conteiner ls -aq` lists all of the containers id's, which is handy since now I can combine it with the rm -f command to remove all of them at once: `docker container rm -f $(docker containers ls -aq)`

---

<div align="center">

## $\color{goldenrod}{\textrm{1.1 - Dockerfile}}$
</div>

Let's run a python-based example project that converts distances. For creating the DOckerFile, we need a python base image, the code itself and any dependencies (and pip, to deal with them). Namely, we`ll use:


```
click==8.0.1
Flask==2.0.1
gunicorn==20.1.0
itsdangerous==2.0.1
Jinja2==3.0.1
MarkupSafe==2.0.1
Werkzeug==2.0.1
```

Since this combination is common for lightweight, scalable web services. We _could_ just use Flask, but its built-in development server is designed for development purposes only. While it's great for testing and debugging during development, it lacks the robustness, performance, and features needed for production environments. Gunicorn (or a similar WSGI server) is typically used for production because it offers better performance, stability, security, scalability and integration. It also has better log handling and debugging capabilities.

So far our folder contains:
- a template subfolder with the index.html file
- the app.py file
- the requirements.txt file

Time to containerize! We now create the Dockerfile:

```
FROM python:3.13.0

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . /app
EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

- It is based on a python image (since that's the core of the app). Remember to always define the tag i.e. the version you're using.
- The `WORKDIR` creates (and enters) an app folder, as if I were to run mkdir + cd.
- With the base image and the folder structure all set, we need to download the project dependencies with pip. Pip needs the requirements.txt, so we need to copy this file to the /app directory! Since we've already "cd"ed into the /app folder, we copy requirements.txt into ".", the current directory. We can simply state requirements.txt because that file is on the same level as the DockerFile. _One ought to be careful with these details, lest one breaks the entire application like a fool of a Took_.
- The `RUN` command tells ~~Pippin~~ pip to install the requirements. The "-r" indicates that the input is a requirements file, not a single package name.
- The `COPY . /app` transfers all of my local files to the container, so it's readily available. I could also write `COPY . .`
- The `EXPOSE 5000` declares that I'm using the port 5000 in the container.
- `CMD ["gunicorn, "--bind", "0.0.0.0:5000", "app:app"]` is only run when the container initializes, _not during the creation of the image, like the previous commands_. Whenever I create a container, I need to specify the initialization commands. `0.0.0.0:5000` states that anyone can perform requisitions to the 5000 port.

Finally, I can cd into the directory with the Dockerfile and run it with

`docker build -t distance-conversion -f Dockerfile .`

- -t is a tag to specify the name of the image;
- -f specifies the file that I'm using to create the image;
- . specifies the context i.e. which is the local directory in my machine that will be sent to the Docker Daemon so it can create the image.

it was successfully created! So now we can run the container with `docker container run -d -p 8181:5000 distance-conversion`, check that it's running with `docker ps` and see the application running on the brower with localhost:8181

![alt text](dc.png)

---

## $\color{goldenrod}{\textrm{1.2 - Docker Registry}}$
</div>

We'll store our public images in DockerHub, the oficial registry from Docker. This service is free for public images, but we can purchase secure storage for private images if needed.

To upload our image to DockerHub, we must follow some standard notation: namespace/repository:tag e.g. hbatistuzzo/distance-conversion:v1

We'll rerun the building with `docker build -t hbatistuzzo/distance-conversion:v1 .`

To push the image to DockerHub, we also need to authentice our login, so `docker login`, input credentials, and then use `docker push hbatistuzzo/distance-conversion:v1`

Since this is the latest version, it's also a good practice to label it as such. Just use `docker tag hbatistuzzo/distance-conversion:v1 hbatistuzzo/distance-conversion:latest`, and then push that with `docker push hbatistuzzo/distance-conversion:latest`

---

![Abhinandan Trilokia](https://raw.githubusercontent.com/Trilokia/Trilokia/379277808c61ef204768a61bbc5d25bc7798ccf1/bottom_header.svg)
