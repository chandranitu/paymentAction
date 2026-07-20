name: Payment Service CI

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
