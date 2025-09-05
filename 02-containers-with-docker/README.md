# 🐳 Docker Lab: From Basic to Advanced

This set of hands-on labs was created to teach you how to use Docker efficiently in daily development and automation. We'll start with basic commands and evolve to troubleshooting, file copying, creating custom images, and using `docker-compose`.

 

## Prerequisites

- Docker installed ([Instructions](https://docs.docker.com/get-docker/))
- Linux terminal or WSL/Mac

Check installed version:
```bash
docker --version
```

Check if the service is active:
```bash
sudo systemctl status docker
```

 

## Docker Lab

### 🎯 Objective

Execute basic Docker commands without losing SSH terminal access

 

## Lab 1 – Basic Commands with Remote Security

### 1. Download Nginx image

```bash
docker pull nginx
```

### 2. Run container in detached mode

```bash
docker run -d --name webserver -p 8080:80 nginx
```

### 3. Check running containers

```bash
docker ps
```

### 4. Check if it's working

```bash
curl http://localhost:8080
```

### 5. Access the container terminal

```bash
docker exec -it webserver bash
```

### 6. Stop and remove the container

```bash
docker stop webserver
docker rm webserver
```

### 7. Remove the image (optional)

```bash
docker rmi nginx
```

## Lab 2 – Docker with Python and FastAPI

### 1. Create project structure

```bash
mkdir -p ~/labs/docker/fastapi-app
cd ~/labs/docker/fastapi-app
```

* Creates the main application directory.

### 2. Create application files

#### Create the main file

```bash
touch main.py
```

#### Content of `main.py`

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello from a Dockerized FastAPI app!"}
```

### 3. Create the `requirements.txt` file

```bash
echo -e "fastapi\nuvicorn" > requirements.txt
```

* Defines the application dependencies.


### 4. Create the Dockerfile

```Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 5. Build the image

```bash
docker build -t fastapi-app .
```

### 6. Run the container

```bash
docker run -d -p 8000:8000 fastapi-app
```

### 7. Test and view logs

#### Test via curl:

```bash
curl http://localhost:8000/
```

* Expected return:

```json
{"message":"Hello from a Dockerized FastAPI app!"}
```

#### View logs:

```bash
docker logs $(docker ps -q --filter ancestor=fastapi-app)
```

### 8. Stop and remove the container (optional)

```bash
docker ps -q --filter ancestor=fastapi-app | xargs docker stop | xargs docker rm
```


## Lab 3 – Docker Multistage with Node.js

### 1. Create project structure

```bash
mkdir -p ~/labs/docker/app
cd ~/labs/docker/app
```

* `mkdir -p` creates the main project directory.
* `cd` navigates to the main directory.


### 2. Create application files

#### Create the directory and main file:

```bash
mkdir -p src
touch src/index.js
```

* `mkdir -p src` ensures the `src` directory exists.
* `touch src/index.js` creates the main application file.

#### Edit the content of `index.js`:

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from a professional Docker container!');
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
```

* This code creates a simple Node.js web server that responds with plain text.


### 3. Create the `package.json` file

#### Create the file:

```bash
touch package.json
```

#### Add the content below to `package.json`:

```json
{
  "name": "docker-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {}
}
```
- Folder structure
```bash
➜  tree
.
├── dockerfile
└── src
    ├── index.js
    └── package.json
```

* This file defines the basic application information and the start command.

### 4. Create optimized Dockerfile

```Dockerfile
# Stage 1: build the application (dependency installation)
FROM node:18 AS builder
WORKDIR /app
COPY src/package.json ./
RUN npm install
COPY src/ .

# Stage 2: final image
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app .
EXPOSE 3000
CMD ["npm", "start"]
```

### 5. Build the image

```bash
docker build -t devopsautomation:latest .
```

### 6. Run the image

```bash
docker run -d -p 3000:3000 devopsautomation:latest
```

### 7. Validate and view logs

#### Test the application via curl:

```bash
curl http://localhost:3000
```

* Expected return is: `Hello from a professional Docker container!`

#### View container logs:

```bash
docker logs $(docker ps -q --filter ancestor=devopsautomation:latest)
```

* Should display something like:

```bash
Servidor rodando na porta 3000
```

### 8. Stop and remove the container (optional)

```bash
docker ps -q --filter ancestor=devopsautomation:latest | xargs docker stop | xargs docker rm
```

## Lab 4 – Creating a .gitignore for Docker Projects with Node.js

### 1. Reusing the Existing Project

Let's take advantage of the project created in **Lab 3 – Docker Multistage with Node.js**. Navigate to the project's base directory:

```bash
cd ~/labs
```

Initialize the repository at the root (if it doesn't exist yet):

```bash
git init
```

> Important: the entire project structure is in `docker/app/`

### 2. Create the `.gitignore` file at the repository root

```bash
touch .gitignore
```

Add the rules considering the project structure:

```gitignore
# Ignore sensitive and temporary files inside docker/app
/docker/app/node_modules/
/docker/app/dist/
/docker/app/*.log
/docker/app/.env
/docker/app/.env*

# Pycache, logs and temporary files in general
**/__pycache__/
**/*.pyc
.DS_Store
*.tgz
```

### 3. Test .gitignore behavior

Create files simulating a real environment:

```bash
mkdir docker/app/node_modules
mkdir docker/app/dist
echo "segredo=abc" > docker/app/.env
```

Check with `git status`:

```bash
git status
```

* The created files/folders should not appear as "Untracked files"

### 4. Add and version only necessary files

```bash
git add .
git status
```

Confirm that only relevant files are being versioned, such as:

* `.gitignore`
* `docker/app/dockerfile`
* `docker/app/src/index.js`
* `docker/app/src/package.json`

```bash
git commit -m "Initial configuration with .gitignore for Docker/Node structure"
```

### 5. Check ignored files

```bash
git check-ignore -v docker/app/.env docker/app/node_modules/
```

## Lab 8 – Pushing Docker Image to Docker Hub

### 1. Prerequisites

* Have a [Docker Hub](https://hub.docker.com/) account
* Have logged in locally with Docker CLI:

```bash
docker login
```

### 2. Reuse the project image

Let's use the image created in **Lab 3 – Docker Multistage with Node.js**

Access the project folder:

```bash
cd ~/labs/docker/app
```

### 3. Tag the image with the remote repository name

The pattern is:

```
<username-dockerhub>/<repository-name>:<tag>
```

Example:

```bash
docker tag devopsautomation:latest iesodias/docker-node-app:1.0.0
```

### 4. Push the image

```bash
docker push iesodias/docker-node-app:1.0.0
```

If you want to send as `latest` as well:

```bash
docker tag devopsautomation:latest iesodias/docker-node-app:latest
docker push iesodias/docker-node-app:latest
```

### 5. Check on Docker Hub

Access your repository on Docker Hub and confirm the tags appear correctly.


### 6. Download from anywhere (simulating remote environment)

On any host with Docker installed:

```bash
docker pull iesodias/docker-node-app:1.0.0
docker run -p 3000:3000 iesodias/docker-node-app:1.0.0
```

### 7. Tip: Remove local image (simulates clean CI/CD)

```bash
docker rmi iesodias/docker-node-app:1.0.0
```

## Lab 9 – Orchestrating with Docker Compose

### 1. Objective

Use Docker Compose to run a Node.js application together with a PostgreSQL database, simulating a complete development environment.

### 2. Project structure

Building on the project from **Lab 3**, let's use this structure:

```
docker-compose-lab/
├── docker-compose.yml
└── app/
    ├── dockerfile
    └── src/
        ├── index.js
        └── package.json
```

### 3. Create the `docker-compose.yml`

At the root of the `docker-compose-lab/` project, create the file:

```yaml
version: '3.8'

services:
  node-app:
    build:
      context: ./app
      dockerfile: dockerfile
    ports:
      - "3000:3000"
    container_name: node_compose_app
    restart: unless-stopped
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://devuser:devpass@db:5432/devdb

  db:
    image: postgres:15
    container_name: postgres_compose_db
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: devpass
      POSTGRES_DB: devdb
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### 4. Start the application with Docker Compose

```bash
docker-compose up -d
```

### 5. Validate application functionality

```bash
curl http://localhost:3000
```

Expected return:

```text
Hello from a professional Docker container!
```

### 6. Access the database

```bash
docker exec -it postgres_compose_db psql -U devuser -d devdb
```

Example command inside PostgreSQL:

```sql
\l  -- list databases
\dt -- list tables
```

### 7. Check container status

```bash
docker-compose ps
```

### 8. View application logs

```bash
docker-compose logs -f node-app
```

### 9. Stop and remove containers

```bash
docker-compose down -v
```

This lab shows how to use Docker Compose to orchestrate a Node.js application with PostgreSQL, preparing a solid foundation for modern development environments.

 