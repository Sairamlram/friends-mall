pipeline {
    agent any

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Source Files') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    if (!(Test-Path "index.html")) {
                        throw "index.html is missing"
                    }

                    if (!(Test-Path "Dockerfile")) {
                        throw "Dockerfile is missing"
                    }

                    Write-Host "FRIENDS Mall source files found."
                '''
            }
        }

        stage('Validate Website Content') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $html = Get-Content "index.html" -Raw

                    if ($html -notmatch "FRIENDS Mall") {
                        throw "FRIENDS Mall branding not found"
                    }

                    if ($html -notmatch "Mall Store Directory") {
                        throw "Mall directory section not found"
                    }

                    Write-Host "Website content validation passed."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    docker info

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker engine is unavailable"
                    }

                    docker build -t friends-mall:latest .

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image build failed"
                    }
                '''
            }
        }
    }

    post {
        success {
            echo 'FRIENDS Mall CI pipeline completed successfully.'
        }

        failure {
            echo 'FRIENDS Mall pipeline failed. Check Console Output.'
        }
    }
}