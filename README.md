# Ecommerce Website: Angular + SpringBoot

Deployment MindMap
![Deployment MindMap](/DeploymentMindMap.jpg)

# Docker Deoloyment
1. create `Dockerfile` for frontend & backend and build docker images for frontend & backend
   - frontend
     ```
     yarn build --prod
     docker login
     docker build -t hahaha12353/ecomm-frontend:latest .
     docker push hahaha12353/ecomm-frontend:latest
     docker pull hahaha12353/ecomm-frontend:latest
     docker run -d -p 4200:4200 --name frontend hahaha12353/ecomm-frontend:latest
     ```
   - backend
     ```
     mvn clean install
     docker login
     docker build -t hahaha12353/ecomm-backend:latest .
     docker push hahaha12353/ecomm-backend:latest
     docker pull hahaha12353/ecomm-backend:latest
     docker run --network="host" hahaha12353/ecomm-backend
     ```
     error:
    ```
    org.postgresql.util.PSQLException: Connection to localhost:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
    
    > Solution:

    ```
   - check network: `docker network ls`
     
2. create `docker-compose.yml`, build frontend & backend in the container
    ```
     docker-compose up
    ```


# Github Action CICD
- edit the `.github/workflows/maven.yml`
  ```
  name: Java CI with Maven

  on:
    push:
      branches: [ "main" ]
    pull_request:
      branches: [ "main" ]

  jobs:
    build:

      runs-on: ubuntu-latest

      services:
        db:
          image: postgres:latest
          env:
            POSTGRES_USER: ${{ secrets.POSTGRES_USER }}
            POSTGRES_PASSWORD: ${{ secrets.POSTGRES_PASSWORD }}
            POSTGRES_DB: e-comm02
          ports:
            - 5432:5432
          options: >-
            --health-cmd pg_isready
            --health-interval 10s
            --health-timeout 5s 
            --health-retries 5    

      steps:
      - uses: actions/checkout@v2

      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'
          cache: maven

      - name: Install PostgreSQL client
        run: sudo apt-get -yqq install postgresql-client

      - name: Check PostgreSQL Connection
        run: |
          PGPASSWORD=${{ secrets.POSTGRES_PASSWORD }} psql -h localhost -U ${{ secrets.POSTGRES_USER }} -d e-comm02 -c 'SELECT 1'

      - name: Build with Maven
        run: mvn clean install
        env:
          DATABASE_USER: ${{ secrets.POSTGRES_USER }}
          DATABASE_PASSWORD: ${{ secrets.POSTGRES_PASSWORD }}
          DATABASE_URL: jdbc:postgresql://localhost:5432/e-comm02 # ${{ secrets.DATABASE_URL }}

      - name: Test with Maven
        run: mvn test
        env:
          DATABASE_URL: jdbc:postgresql://localhost:5432/e-comm02 # ${{ secrets.DATABASE_URL }}

      - name: Deploy to GitHub Releases (Simple Example)
        if: startsWith(github.ref, 'refs/tags/')
        uses: actions/upload-release-asset@v1
        with:
          upload_url: ${{ github.event.release.upload_url }}
          asset_path: ./target/ecomm-0.0.1-SNAPSHOT.jar # replace with your artifact's path
          asset_name: ecomm-0.0.1-SNAPSHOT.jar
          asset_content_type: application/jar

      - name: Build and push docker image
        uses: mr-smithers-excellent/docker-build-push@v6
        with:
          image: hahaha12353/backend
          tags: latest
          registry: docker.io
          dockerfile: Dockerfile
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}


  ```


# Kubernetes Deployment

1. start kubernetes
   > start docker <br>
   > minikube start
   >  minikube dashboard
  
2. goto `/backend` path -> edit `postgres-deployment.yml` and `backend-deployment.yml` -> deploy
    ```
    mvn clean install
    docker login
    docker build -t hahaha12353/backend:latest .
    docker push hahaha12353/backend:latest

    kubectl apply -f postgres-deployment.yaml
    kubectl apply -f backend-deployment.yml
    minikube service backend-service

    ===
    > minikube service backend-service
    "host-url": "http://127.0.0.1:12273/api",

    ===
    > kubectl port-forward service/backend-service 9002:9002
    "host-url": "http://localhost:9002/api",

    kubectl logs deployment/backend-deployment

    kubectl describe pod backend-deployment-754bd9499b-9dt9l

    ```
3. goto `/frontend` path -> edit `frontend-deployment.yml` -> deploy
    ```
    yarn build --prod
    docker build -t hahaha12353/frontend:latest .
    docker push hahaha12353/frontend:latest

    kubectl apply -f frontend-deployment.yaml

    minikube service frontend-service

    kubectl get service frontend-service

    kubectl logs frontend-deployment

    kubectl delete pod frontend-deployment-59ccc7ffb9-sfbsh
    ```