name: Payment Service CI/CD - Azure

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'
          cache: maven

      - name: Build Project
        run: mvn clean compile

      - name: Run Unit Tests
        run: mvn test

      - name: Package Application
        run: mvn clean package -DskipTests

      - name: Upload JAR
        uses: actions/upload-artifact@v4
        with:
          name: payment-service
          path: target/*.jar

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker Image
        run: docker build -t ${{ secrets.DOCKER_USERNAME }}/payment-service:latest .

      - name: Push Docker Image
        run: docker push ${{ secrets.DOCKER_USERNAME }}/payment-service:latest

      - name: Deploy to Azure VM
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.AZURE_VM_HOST }}
          username: ${{ secrets.AZURE_VM_USER }}
          key: ${{ secrets.AZURE_VM_SSH_KEY }}
          script: |
            # Pull latest application image
            docker pull ${{ secrets.DOCKER_USERNAME }}/payment-service:latest

            # Create Docker network if it doesn't exist
            docker network inspect payment-network >/dev/null 2>&1 || docker network create payment-network

            # Stop and remove MongoDB
            docker stop mongodb || true
            docker rm mongodb || true

            # Start MongoDB
            docker run -d \
              --name mongodb \
              --network payment-network \
              --restart always \
              -p 27017:27017 \
              -e MONGO_INITDB_ROOT_USERNAME=admin \
              -e MONGO_INITDB_ROOT_PASSWORD=admin123 \
              mongo:latest

            # Stop and remove Payment Service
            docker stop payment-service || true
            docker rm payment-service || true

            # Start Payment Service
            docker run -d \
              --name payment-service \
              --network payment-network \
              --restart always \
              -p 8080:8080 \
              -e SPRING_DATA_MONGODB_URI="mongodb://admin:admin123@mongodb:27017/paymentdb?authSource=admin" \
              ${{ secrets.DOCKER_USERNAME }}/payment-service:latest
