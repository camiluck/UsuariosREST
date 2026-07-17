pipeline {
    agent any

    environment {
        IMAGE_NAME = "usuarios-rest"
        CONTAINER_NAME = "usuarios-rest-app"
        APP_PORT = "8081" // puerto externo (deja 8080 libre para Jenkins)
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Clonando el repositorio desde GitHub...'
                git branch: 'main', url: 'https://github.com/camiluck/UsuariosREST.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Compilando el proyecto con Maven...'
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo 'Ejecutando pruebas unitarias...'
                withCredentials([
                    string(credentialsId: 'db-host', variable: 'DB_HOST'),
                    string(credentialsId: 'db-user', variable: 'DB_USER'),
                    string(credentialsId: 'db-password', variable: 'DB_PASSWORD')
                ]) {
                    sh './mvnw test'
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Construyendo la imagen Docker...'
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Desplegando el contenedor...'
                // Requiere crear en Jenkins: Manage Jenkins > Credentials > Add Credentials
                // - Secret text con id "db-host"     -> endpoint de RDS
                // - Secret text con id "db-user"     -> usuario de RDS
                // - Secret text con id "db-password" -> password de RDS
                withCredentials([
                    string(credentialsId: 'db-host', variable: 'DB_HOST'),
                    string(credentialsId: 'db-user', variable: 'DB_USER'),
                    string(credentialsId: 'db-password', variable: 'DB_PASSWORD')
                ]) {
                    sh '''
                        docker rm -f $CONTAINER_NAME || true
                        docker run -d --name $CONTAINER_NAME \
                          -p $APP_PORT:8080 \
                          -e DB_HOST=$DB_HOST \
                          -e DB_USER=$DB_USER \
                          -e DB_PASSWORD=$DB_PASSWORD \
                          $IMAGE_NAME:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline ejecutado correctamente. App disponible en el puerto ' + env.APP_PORT
        }
        failure {
            echo 'El pipeline falló. Revisar el Console Output de la etapa fallida.'
        }
        always {
            sh 'docker ps --filter "name=$CONTAINER_NAME"'
        }
    }
}
