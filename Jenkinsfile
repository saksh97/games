pipeline {
    agent { label 'agent1' } 

    parameters {
        choice(
            name: 'CHOSEN_GAME', 
            choices: ['2048_Puzzle_Game', 'Tic_Tac_Toe'], 
            description: 'Select which retro browser game you want to deploy to your Nginx container.'
        )
    }

    environment {
        // Core game repository links provided by you
        URL_2048 = 'https://github.com/gabrielecirulli/2048.git'
        URL_TIC_TAC_TOE = 'https://github.com/rammijar/Tic-tac-toe.git'
        NGINX_CONTAINER_NAME = 'my-nginx-container'
    }

    stages {
        stage('Verify & Install Docker Engine') {
            steps {
                echo 'Ensuring system tools and docker engine are operational on agent1...'
                sh '''
                    sudo apt-get update -qq
                    sudo apt-get install -y git docker.io
                '''
            }
        }

        stage('Checkout Game Code') {
            steps {
                script {
                    echo 'Cleaning workspace staging directories...'
                    cleanWs()
                    
                    // Dynamically clone based on the parameter choice dropdown selection
                    if (params.CHOSEN_GAME == '2048_Puzzle_Game') {
                        echo "Fetching 2048 Puzzle Game code from master branch..."
                        git branch: 'master', url: "${env.URL_2048}"
                    } else if (params.CHOSEN_GAME == 'Tic_Tac_Toe') {
                        echo "Fetching Tic-Tac-Toe Game code from main branch..."
                        git branch: 'main', url: "${env.URL_TIC_TAC_TOE}"
                    }
                }
            }
        }

        stage('Deploy to Nginx Container') {
            steps {
                echo "Managing container lifecycle and deploying ${params.CHOSEN_GAME}..."
                sh """
                    # 1. Force remove any existing container layers to completely flush the cache
                    if [ \$(sudo docker ps -a -q -f name=^/${env.NGINX_CONTAINER_NAME}\$) ]; then
                        echo "Removing active container layers to clear asset memory cache..."
                        sudo docker rm -f ${env.NGINX_CONTAINER_NAME}
                    fi

                    # 2. Spin up a clean Nginx Alpine container on host port 81
                    echo "Spinning up a fresh container instance..."
                    sudo docker run -d -p 81:80 --name ${env.NGINX_CONTAINER_NAME} nginx:alpine
                    sleep 3

                    # 3. Clear default container template welcome landing pages
                    sudo docker exec ${env.NGINX_CONTAINER_NAME} rm -rf /usr/share/nginx/html/*
                    
                    # 4. Copy the freshly cloned workspace game assets directly into the container filesystem
                    sudo docker cp . ${env.NGINX_CONTAINER_NAME}:/usr/share/nginx/html/
                    
                    # 5. Reset access controls inside the container workspace
                    sudo docker exec ${env.NGINX_CONTAINER_NAME} chmod -R 755 /usr/share/nginx/html
                """
                echo 'Game environment hot-swap completed successfully!'
            }
        }
    }
    
    post {
        success {
            echo "------------------------------------------------------------"
            echo "SUCCESS: ${params.CHOSEN_GAME} is now live and deployable!"
            echo "Play the game at your new EC2 IP: http://13.201.193.8:81"
            echo "------------------------------------------------------------"
        }
        failure {
            echo "ERROR: Pipeline step failed. Please check the logs above."
        }
    }
}
