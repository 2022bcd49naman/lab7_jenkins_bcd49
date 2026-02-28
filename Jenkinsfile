pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "2022bcd0049namanomar/wine-quality:latest"
        CONTAINER_NAME = "wine_test_container_bcd49_namanomar"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh 'docker pull ${DOCKER_IMAGE}'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f ${CONTAINER_NAME} || true
                docker run -d --name ${CONTAINER_NAME} ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Wait for API Readiness') {
            steps {
                sh '''
                for i in {1..15}; do
                    echo "Checking API..."
                    sleep 5
                    if docker exec ${CONTAINER_NAME} curl -s http://localhost:8000/health | grep "ok"; then
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
                RESPONSE=$(docker exec ${CONTAINER_NAME} curl -s -X POST http://localhost:8000/predict \
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
                STATUS=$(docker exec ${CONTAINER_NAME} curl -o /dev/null -s -w "%{http_code}" \
                -X POST http://localhost:8000/predict \
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
    }

    post {
        always {
            sh '''
            docker rm -f ${CONTAINER_NAME} || true
            '''
        }
    }
}