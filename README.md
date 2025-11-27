In this DevOps task, you need to build and deploy a full-stack CRUD application using the MEAN stack (MongoDB, Express, Angular 15, and Node.js). The backend will be developed with Node.js and Express to provide REST APIs, connecting to a MongoDB database. The frontend will be an Angular application utilizing HTTPClient for communication.  

The application will manage a collection of tutorials, where each tutorial includes an ID, title, description, and published status. Users will be able to create, retrieve, update, and delete tutorials. Additionally, a search box will allow users to find tutorials by title.

## Project setup

### Node.js Server

cd backend

npm install

You can update the MongoDB credentials by modifying the `db.config.js` file located in `app/config/`.

Run `node server.js`

### Angular Client

cd frontend

npm install

Run `ng serve --port 8081`

You can modify the `src/app/services/tutorial.service.ts` file to adjust how the frontend interacts with the backend.

Navigate to `http://localhost:8081/`

-----------------------------------------------------------------------------------------------------------------------------------------

Steps :

Project Structure :

crud-dd-task-mean-app/
├── backend/ # Node.js + Express API
│ ├── Dockerfile
│ └── ...
├── frontend/ # Angular application
│ ├── Dockerfile
│ └── ...
├── docker-compose.yml
├── .github/workflows/ci-cd.yml
└── README.md


Prerequisites :

Git

Docker & Docker Compose installed on your local machine or VM

GitHub account and repository

Docker Hub account

Ubuntu VM on any cloud provider (AWS)



Backend Dockerfile :

FROM node:18-alpine
WORKDIR /usr/src/app
COPY backend/package*.json ./
RUN npm install
COPY backend/ ./
EXPOSE 8080
CMD ["npm", "start"]


frontend Dockerfile :

# Build Stage
FROM node:18-alpine AS build
WORKDIR /app
COPY frontend/package*.json ./
RUN npm install
COPY frontend/ .
RUN npm run build --prod

# Nginx Stage
FROM nginx:stable-alpine
RUN rm -rf /usr/share/nginx/html/*
COPY --from=build /app/dist/* /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]


Build & Push Docker Images :

      - name: Build and push backend image
        uses: docker/build-push-action@v4
        with:
          context: .
          file: backend/Dockerfile.backend
          push: true
          tags: techhubdevops/my-docker:backend-latest

      - name: Build and push frontend image
        uses: docker/build-push-action@v4
        with:
          context: .
          file: frontend/Dockerfile.frontend
          push: true
          tags: techhubdevops/my-docker:frontend-latest
		  

Docker Compose Deployment :

version: '3.8'

services:
  mongo:
    image: mongo:6.0
    restart: unless-stopped
    environment:
      MONGO_INITDB_DATABASE: meanapp
    volumes:
      - mongo_data:/data/db

  backend:
    build:
      context: .
      dockerfile: backend/Dockerfile.backend
    image: yourdockerhub/mean-backend:latest
    environment:
      - PORT=8080
      - MONGODB_URI=mongodb://mongo:27017/meanapp
    depends_on:
      - mongo
    expose:
      - "8080"
    restart: unless-stopped

  frontend:
    build:
      context: .
      dockerfile: frontend/Dockerfile.frontend
    image: yourdockerhub/mean-frontend:latest
    expose:
      - "80"
    restart: unless-stopped

  nginx-proxy:
    image: nginx:stable-alpine
    volumes:
      - ./nginx-proxy/default.conf:/etc/nginx/conf.d/default.conf:ro
    ports:
      - "80:80"
    depends_on:
      - frontend
      - backend
    restart: unless-stopped

volumes:
  mongo_data:


github/workflows/ci-cd :-

name: CI/CD Pipeline


on:
push:
branches:
- main


jobs:
build-and-deploy:
runs-on: ubuntu-latest


steps:
- uses: actions/checkout@v3


- name: Set up Docker
uses: docker/setup-buildx-action@v2


- name: Login to Docker Hub
uses: docker/login-action@v2
with:
username: ${{ secrets.DOCKER_USERNAME }}
password: ${{ secrets.DOCKER_PASSWORD }}


- name: Build and push backend image
run: |
docker build -t <dockerhub-username>/mean-backend:latest ./backend
docker push <dockerhub-username>/mean-backend:latest


- name: Build and push frontend image
run: |
docker build -t <dockerhub-username>/mean-frontend:latest ./frontend
docker push <dockerhub-username>/mean-frontend:latest


- name: Deploy to VM
uses: appleboy/ssh-action@v0.1.7
with:
host: ${{ secrets.VM_HOST }}
username: ${{ secrets.VM_USER }}
key: ${{ secrets.VM_SSH_KEY }}
script: |
cd /home/ubuntu/<project-folder>
docker-compose pull
docker-compose up -d


Nginx Reverse Proxy :

server {
listen 80;
server_name _;


# API proxy to backend
location /api/ {
proxy_pass http://backend:8080/api/;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection 'upgrade';
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
}


# Root (frontend static)
location / {
proxy_pass http://frontend:80/;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
}



Website URL :-

http://18.60.226.180/add
http://18.60.226.180/tutorial
