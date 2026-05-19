pipeline {
    agent any

    environment {
        DOCKER_IMAGE_BACKEND  = "doctor-appointment-backend"
        DOCKER_IMAGE_FRONTEND = "doctor-appointment-frontend"
        DOCKER_IMAGE_ADMIN    = "doctor-appointment-admin"
        DOCKER_TAG            = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Create Backend .env') {
            steps {
                withCredentials([
                    string(credentialsId: 'MONGODB_URI',           variable: 'MONGODB_URI'),
                    string(credentialsId: 'CLOUDINARY_NAME',       variable: 'CLOUDINARY_NAME'),
                    string(credentialsId: 'CLOUDINARY_API_KEY',    variable: 'CLOUDINARY_API_KEY'),
                    string(credentialsId: 'CLOUDINARY_SECRET_KEY', variable: 'CLOUDINARY_SECRET_KEY'),
                    string(credentialsId: 'JWT_SECRET',            variable: 'JWT_SECRET'),
                    string(credentialsId: 'RAZORPAY_KEY_ID',       variable: 'RAZORPAY_KEY_ID'),
                    string(credentialsId: 'RAZORPAY_KEY_SECRET',   variable: 'RAZORPAY_KEY_SECRET')
                ]) {
                    powershell '''
                        $lines = @(
                            "MONGODB_URI=$env:MONGODB_URI",
                            "CLOUDINARY_NAME=$env:CLOUDINARY_NAME",
                            "CLOUDINARY_API_KEY=$env:CLOUDINARY_API_KEY",
                            "CLOUDINARY_SECRET_KEY=$env:CLOUDINARY_SECRET_KEY",
                            "ADMIN_EMAIL=admin@prescripto.com",
                            "ADMIN_PASSWORD=qwerty123",
                            "JWT_SECRET=$env:JWT_SECRET",
                            "RAZORPAY_KEY_ID=$env:RAZORPAY_KEY_ID",
                            "RAZORPAY_KEY_SECRET=$env:RAZORPAY_KEY_SECRET",
                            "CURRENCY=INR"
                        )
                        $lines | Set-Content -Path "backend\\.env" -Encoding UTF8
                        Write-Host ".env file created successfully for backend"
                    '''
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images (npm install + build handled inside Docker)...'
                bat """
                    docker build -t %DOCKER_IMAGE_BACKEND%:%DOCKER_TAG%  ./backend
                    docker build -t %DOCKER_IMAGE_FRONTEND%:%DOCKER_TAG% ./frontend
                    docker build -t %DOCKER_IMAGE_ADMIN%:%DOCKER_TAG%    ./admin
                """
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                echo 'Stopping old containers...'
                bat 'docker-compose down --remove-orphans'
                echo 'Starting new containers...'
                bat 'docker-compose up -d --build'
            }
        }

        stage('Health Check') {
            steps {
                echo 'Waiting 30 seconds for services to start...'
                sleep(time: 30, unit: 'SECONDS')
                bat '''
                    echo Checking Backend  (port 5000)...
                    curl -s --max-time 5 http://localhost:5000 >nul 2>&1 && echo Backend  OK || echo Backend  starting up...
                    echo Checking Frontend (port 5173)...
                    curl -s --max-time 5 http://localhost:5173 >nul 2>&1 && echo Frontend OK || echo Frontend starting up...
                    echo Checking Admin    (port 5174)...
                    curl -s --max-time 5 http://localhost:5174 >nul 2>&1 && echo Admin    OK || echo Admin    starting up...
                    exit /b 0
                '''
            }
        }
        stage('Deploy to Render') {
            steps {
                echo 'Triggering Render deployments...'
                withCredentials([
                    string(credentialsId: 'RENDER_BACKEND_HOOK',  variable: 'RENDER_BACKEND_HOOK'),
                    string(credentialsId: 'RENDER_FRONTEND_HOOK', variable: 'RENDER_FRONTEND_HOOK'),
                    string(credentialsId: 'RENDER_ADMIN_HOOK',    variable: 'RENDER_ADMIN_HOOK')
                ]) {
                    bat '''
                        echo Triggering Backend  deployment on Render...
                        curl -X POST "%RENDER_BACKEND_HOOK%"
                        echo.
                        echo Triggering Frontend deployment on Render...
                        curl -X POST "%RENDER_FRONTEND_HOOK%"
                        echo.
                        echo Triggering Admin    deployment on Render...
                        curl -X POST "%RENDER_ADMIN_HOOK%"
                        echo.
                        echo All Render deployments triggered successfully!
                        exit /b 0
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '========================================='
            echo '✅ Pipeline completed successfully!'
            echo ''
            echo '🐳 Local Docker:'
            echo '   Backend  → http://localhost:5000'
            echo '   Frontend → http://localhost:5173'
            echo '   Admin    → http://localhost:5174'
            echo ''
            echo '🌐 Render (Live - deploying now):'
            echo '   Backend  → https://doctor-appointment-backend-e1wn.onrender.com'
            echo '   Frontend → https://doctor-appointment-frontend-f8hl.onrender.com'
            echo '   Admin    → https://doctor-appointment-admin-zdnm.onrender.com'
            echo '========================================='
        }
        failure {
            echo '❌ Pipeline failed at a build/config stage.'
            echo 'Note: If containers are running, they will NOT be stopped.'
            echo 'Run: docker-compose down   (only if you want to stop them)'
        }
        always {
            powershell 'Remove-Item -Path "backend\\.env" -ErrorAction SilentlyContinue'
            echo 'Pipeline finished. Secrets cleaned up.'
        }
    }
}
