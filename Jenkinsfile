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

        stage('Validate Website') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    if (!(Test-Path "index.html")) {
                        throw "index.html is missing"
                    }

                    if (!(Test-Path "Dockerfile")) {
                        throw "Dockerfile is missing"
                    }

                    $html = Get-Content "index.html" -Raw

                    if ($html -notmatch "FRIENDS Mall") {
                        throw "FRIENDS Mall branding not found"
                    }

                    Write-Host "Website validation passed."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    docker build -t friends-mall:latest .

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image build failed"
                    }
                '''
            }
        }

        stage('Test Docker Image') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    docker rm -f friends-mall-test 2>$null

                    docker run -d --name friends-mall-test `
                        -p 8082:80 friends-mall:latest

                    if ($LASTEXITCODE -ne 0) {
                        throw "Test container failed to start"
                    }

                    try {
                        $ready = $false

                        for ($i = 1; $i -le 15; $i++) {
                            try {
                                $response = Invoke-WebRequest `
                                    -Uri "http://localhost:8082" `
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
                            throw "Website health check failed"
                        }

                        Write-Host "Website test passed."
                    }
                    finally {
                        docker rm -f friends-mall-test 2>$null
                    }
                '''
            }
        }

        stage('Deploy Website') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    docker rm -f friends-mall 2>$null

                    docker run -d --name friends-mall `
                        -p 8080:80 friends-mall:latest

                    if ($LASTEXITCODE -ne 0) {
                        throw "Website deployment failed"
                    }

                    Write-Host "FRIENDS Mall deployed."
                '''
            }
        }
    }

    post {
        success {
            echo 'FRIENDS Mall pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}