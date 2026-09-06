# myapp

Simple Node.js + Express web app, containerized with Docker and auto-deployed to an AWS EC2 server using GitHub Actions.

## Run locally
npm install
npm start
# open http://localhost:3000

## Run with Docker
docker build -t myapp .
docker run -p 3000:3000 myapp

## Deploy
Every push to `main` triggers `.github/workflows/deploy.yml`, which builds the image,
pushes it to Docker Hub, and restarts the container on the EC2 server.
