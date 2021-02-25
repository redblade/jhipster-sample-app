#!/usr/bin/env groovy

node {
    APP_NAME = "jhipstersampleapp"
    BRANCH_NAME = "master"
    DOCKER_IMAGE_TAG = "$APP_NAME:R${env.BUILD_ID}"

    
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
        sh "java -version"
    }

    stage('clean') {
        sh "chmod +x mvnw"
        sh "./mvnw -ntp clean -P-webpack"
    }
    stage('nohttp') {
        sh "./mvnw -ntp checkstyle:check"
    }

    stage('install tools') {
        sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm -DnodeVersion=v12.16.1 -DnpmVersion=6.14.5"
    }

    stage('npm install') {
        sh "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
    }

    stage('packaging') {
        sh "./mvnw -ntp verify -P-webpack -Pprod -DskipTests"
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
    }

    stage('publish docker') {
        withCredentials([usernamePassword(credentialsId: 'myregistry-login', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
            sh "echo ${USERNAME} ${PASSWORD} ${DOCKER_IMAGE_TAG}"
            sh "docker login -u ${USERNAME} -p ${PASSWORD}"
            sh "./mvnw -ntp jib:build -Dimage=$DOCKER_IMAGE_TAG"
            sh "docker push ${USERNAME}/${DOCKER_IMAGE_TAG}"
        }
    }
}
