pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        IMAGE_NAME = 'friends-mall:latest'
        CONTAINER_NAME = 'friends-mall'
        HOST_PORT = '8080'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Files') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    if (!(Test-Path "index.html")) {
                        throw "index.html is missing"
                    }

                    if (!(Test-Path "Dockerfile")) {
                        throw "Dockerfile is missing"
                    }

                    Write-Host "Required files exist."
                '''
            }
        }

        stage('Validate Website') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $html = Get-Content "index.html" -Raw

                    if ($html -notmatch "FRIENDS Mall") {
                        throw "FRIENDS Mall branding not found"
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

        stage('Deploy Container') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $existing = docker ps -aq --filter "name=^friends-mall$"

                    if ($existing) {
                        docker rm -f friends-mall

                        if ($LASTEXITCODE -ne 0) {
                            throw "Could not remove the existing container"
                        }
                    }

                    docker run -d --name friends-mall -p 8080:80 friends-mall:latest

                    if ($LASTEXITCODE -ne 0) {
                        throw "Container deployment failed"
                    }
                '''
            }
        }

        stage('Test Website') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    $ready = $false

                    for ($i = 1; $i -le 15; $i++) {
                        try {
                            $response = Invoke-WebRequest `
                                -Uri "http://localhost:8080" `
                                -TimeoutSec 3

                            if ($response.StatusCode -eq 200 -and
                                $response.Content -match "FRIENDS Mall") {
                                $ready = $true
                                break
                            }
                        }
                        catch {
                            Start-Sleep -Seconds 2
                        }
                    }

                    if (!$ready) {
                        docker logs friends-mall
                        throw "Website health check failed"
                    }

                    Write-Host "FRIENDS Mall website is responding successfully."
                '''
            }
        }
    }

    post {
        success {
            echo 'FRIENDS Mall CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Review Console Output.'
        }
    }
}