pipeline {
    agent any

    environment {
        PROJECT_NAME   = 'wireguard-gateway'
        CONFIG_PATH    = 'config/wg_confs/wg0.conf'
        CONTAINER_NAME = 'wireguard'
    }

    stages {
        stage('📥 Checkout from GitHub') {
            steps {
                echo "=== Cloning repository from GitHub ==="
                checkout scm
                sh 'git log -1 --pretty=format:"%h - %an: %s"'
            }
        }

        stage('🔍 Validate Config Files') {
            steps {
                echo "=== Validating WireGuard config ==="
                script {
                    if (fileExists(env.CONFIG_PATH)) {
                        echo "✅ ${env.CONFIG_PATH} found"
                        sh "head -20 ${env.CONFIG_PATH}"
                    } else {
                        error "❌ ${env.CONFIG_PATH} not found!"
                    }
                }
            }
        }

        stage('🧪 Test Config Syntax') {
            steps {
                echo "=== Testing WireGuard config syntax ==="
                sh '''
                    if grep -q "\\[Interface\\]" config/wg_confs/wg0.conf; then
                        echo "✅ [Interface] section found"
                    else
                        echo "❌ [Interface] section missing!"
                        exit 1
                    fi

                    if grep -q "ListenPort" config/wg_confs/wg0.conf; then
                        echo "✅ ListenPort configured"
                    else
                        echo "⚠️  Warning: ListenPort not configured"
                    fi

                    echo "✅ Config syntax validation passed"
                '''
            }
        }

        stage('💾 Backup Current Config') {
            steps {
                echo "=== Backing up current WireGuard config ==="
                sh """
                    docker exec ${env.CONTAINER_NAME} cp /config/wg_confs/wg0.conf /config/wg_confs/wg0.conf.backup-${BUILD_NUMBER} || true
                    echo "✅ Backup created: wg0.conf.backup-${BUILD_NUMBER}"

                    docker exec ${env.CONTAINER_NAME} ls -lht /config/wg_confs/*.backup* | head -5 || echo "No previous backups"
                """
            }
        }

        stage('🚀 Deploy New Config') {
            steps {
                echo "=== Deploying new config to WireGuard ==="
                sh """
                    docker cp ${env.CONFIG_PATH} ${env.CONTAINER_NAME}:/config/wg_confs/wg0.conf
                    docker exec ${env.CONTAINER_NAME} chmod 600 /config/wg_confs/wg0.conf
                    docker exec ${env.CONTAINER_NAME} chown root:root /config/wg_confs/wg0.conf || true
                    docker exec ${env.CONTAINER_NAME} ls -lh /config/wg_confs/wg0.conf
                    echo "✅ Config deployed successfully"
                """
            }
        }

        stage('🔄 Restart WireGuard') {
            steps {
                echo "=== Restarting WireGuard service ==="
                sh """
                    echo "Stopping WireGuard container..."
                    docker stop ${env.CONTAINER_NAME} || true
                    sleep 5

                    echo "Starting WireGuard container..."
                    docker start ${env.CONTAINER_NAME}
                    echo "⏳ Waiting for WireGuard to start..."
                    sleep 15
                """
            }
        }

        stage('✅ Health Check') {
            steps {
                echo "=== Verifying WireGuard is running ==="
                script {
                    def isRunning = sh(
                        script: "docker inspect -f '{{.State.Running}}' ${env.CONTAINER_NAME} || echo 'false'",
                        returnStdout: true
                    ).trim()

                    if (isRunning == "true") {
                        echo "✅ Container ${env.CONTAINER_NAME} is running"
                    } else {
                        error "❌ Container ${env.CONTAINER_NAME} is not running!"
                    }

                    // pakai retry untuk nunggu wg interface ready
                    retry(3) {
                        sh "docker exec ${env.CONTAINER_NAME} wg show | head -10"
                    }

                    echo "✅ WireGuard interface is active!"
                }
            }
        }
    }
}
