name: Payment Service CI/CD

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

      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ secrets.DOCKER_USERNAME }}/payment-service:latest

            docker stop payment-service || true
            docker rm payment-service || true

            docker run -d \
              --name payment-service \
              -p 8080:8080 \
              --restart always \
              ${{ secrets.DOCKER_USERNAME }}/payment-service:latest
