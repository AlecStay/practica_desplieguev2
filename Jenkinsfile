pipeline {
    agent any

    tools {
        nodejs "Node18"
    }

    stages {
        stage('Instalar dependencias') {
            steps {
                sh 'npm install'
            }
        }

        stage('Ejecutar tests') {
            steps {
                // Agrega esta línea para dar permisos de ejecución:
                sh 'chmod +x -R node_modules/.bin/' 
                sh 'npm test'
            }
        }

        stage('Construir Imagen Docker') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh '''
                    set -eux

                    DOCKER_CLI_VERSION="25.0.5"
                    DOCKER_CLI_DIR="$WORKSPACE/.docker-cli"

                    mkdir -p "$DOCKER_CLI_DIR"

                    if [ ! -x "$DOCKER_CLI_DIR/docker" ]; then
                        ARCH="$(uname -m)"
                        case "$ARCH" in
                            x86_64|amd64) ARCH="x86_64" ;;
                            aarch64|arm64) ARCH="aarch64" ;;
                            *) echo "Arquitectura no soportada: $ARCH"; exit 1 ;;
                        esac

                        URL="https://download.docker.com/linux/static/stable/$ARCH/docker-${DOCKER_CLI_VERSION}.tgz"
                        TMP_TGZ="/tmp/docker.tgz"

                        if command -v curl >/dev/null 2>&1; then
                            curl -fsSL "$URL" -o "$TMP_TGZ"
                        elif command -v wget >/dev/null 2>&1; then
                            wget -qO "$TMP_TGZ" "$URL"
                        else
                            echo "Necesito curl o wget para descargar Docker CLI"; exit 1
                        fi

                        rm -rf /tmp/docker
                        mkdir -p /tmp/docker
                        tar -xzf "$TMP_TGZ" -C /tmp
                        mv /tmp/docker/* "$DOCKER_CLI_DIR/"
                        chmod +x "$DOCKER_CLI_DIR/docker"
                    fi

                    export PATH="$DOCKER_CLI_DIR:$PATH"

                    docker version
                    docker build -t hola-mundo-node:latest .
                '''
            }
        }

        stage('Ejecutar Contenedor Node.js') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh '''
                    set -eux

                    DOCKER_CLI_DIR="$WORKSPACE/.docker-cli"
                    export PATH="$DOCKER_CLI_DIR:$PATH"

                    docker stop hola-mundo-node || true
                    docker rm hola-mundo-node || true
                    docker run -d --name hola-mundo-node -p 3000:3000 hola-mundo-node:latest
                '''
            }
        }
    }
}
 