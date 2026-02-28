pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "2022bcd0049namanomar/wine-quality:latest"
        CONTAINER_NAME = "wine_test_container_bcd49_namanomar"
        PORT = "8000"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh 'docker pull ${DOCKER_IMAGE}'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker run -d -p ${PORT}:8000 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}'
            }
        }

        stage('Wait for API Readiness') {
            steps {
                sh '''
                for i in {1..10}; do
                    sleep 5
                    if curl -s http://localhost:${PORT}/health | grep "ok"; then
                        echo "API Ready"
                        exit 0
                    fi
                done
                echo "API failed to start"
                exit 1
                '''
            }
        }

        stage('Valid Inference Test') {
            steps {
                sh '''
                RESPONSE=$(curl -s -X POST http://localhost:${PORT}/predict \
                -H "Content-Type: application/json" \
                -d '{
                  "fixed_acidity":7.4,
                  "volatile_acidity":0.7,
                  "citric_acid":0,
                  "residual_sugar":1.9,
                  "chlorides":0.076,
                  "free_sulfur_dioxide":11,
                  "total_sulfur_dioxide":34,
                  "density":0.9978,
                  "pH":3.51,
                  "sulphates":0.56,
                  "alcohol":9.4
                }')

                echo "Response: $RESPONSE"

                echo $RESPONSE | grep wine_quality
                '''
            }
        }

        stage('Invalid Inference Test') {
            steps {
                sh '''
                STATUS=$(curl -o /dev/null -s -w "%{http_code}" \
                -X POST http://localhost:${PORT}/predict \
                -H "Content-Type: application/json" \
                -d '{"wrong":"data"}')

                echo "Status Code: $STATUS"

                if [ "$STATUS" -eq 200 ]; then
                    echo "Invalid input test failed"
                    exit 1
                fi
                '''
            }
        }

        stage('Stop Container') {
            steps {
                sh '''
                docker stop ${CONTAINER_NAME}
                docker rm ${CONTAINER_NAME}
                '''
            }
        }
    }
}