#! /usr/bin/env groovy

pipeline {
    agent any
    stages {
        stage('测试构建') {
            steps {
                sh '/home/jenkins/.cargo/bin/mdbook --version'
            }
        }
    }
}
