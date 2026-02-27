// pipeline {
//     agent any

//     environment {
//         VENV_DIR = "venv"
//         METRICS_FILE = "training-artifacts-py3.11/metrics.json"
//         BEST_ACCURACY_FILE = "best-accuracy"
//         DOCKER_IMAGE = "2022bcd0013ashiqfiroz/wine-quality-app-jenkins"
//         CURRENT_ACCURACY = "0"
//         MODEL_IMPROVED = "false"
//         ARTIFACTS_DIR = "training-artifacts-py3.11"
//     }

//     stages {

//         stage('Checkout') {
//             steps {
//                 git branch: 'main',
//                     url: 'https://github.com/2022bcd0013-ashiq-firoz/lab3.git'
//             }
//         }

//         stage('Setup Python Virtual Environment') {
//             steps {
//                 sh '''
//                     python3 -m venv $VENV_DIR
//                     . $VENV_DIR/bin/activate
//                     pip install --upgrade pip
//                     pip install -r requirements.txt
//                 '''
//             }
//         }

//         stage('Train Model') {
//             steps {
//                 sh '''
//                     . $VENV_DIR/bin/activate
                    
//                     # Create output directory for training script
//                     mkdir -p output
                    
//                     # Run training
//                     python Script/train.py
                    
//                     # Copy artifacts from output/ to training-artifacts-py3.11/
//                     mkdir -p training-artifacts-py3.11
//                     cp -r output/* training-artifacts-py3.11/
                    
//                     # Verify copy was successful
//                     echo "Contents of training-artifacts-py3.11/:"
//                     ls -la training-artifacts-py3.11/
//                 '''
//             }
//         }

//         stage('Archive Model Artifacts') {
//             steps {
//                 script {
//                     // Verify files exist before archiving
//                     sh 'ls -la training-artifacts-py3.11/'
                    
//                     // Archive the trained model files
//                     archiveArtifacts artifacts: 'training-artifacts-py3.11/**/*', allowEmptyArchive: false
                    
//                     // Stash for use in later stages
//                     stash includes: 'training-artifacts-py3.11/**/*', name: 'model-artifacts'
                    
//                     echo "Model artifacts archived successfully"
//                 }
//             }
//         }

//         stage('Read Accuracy') {
//             steps {
//                 script {
//                     if (!fileExists(env.METRICS_FILE)) {
//                         echo "WARNING: Metrics file not found. Setting accuracy to 0."
//                         env.CURRENT_ACCURACY = "0"
//                         return
//                     }

//                     def accuracy = sh(
//                         script: "jq -r '.[-1].accuracy' ${METRICS_FILE} 2>/dev/null || echo 0",
//                         returnStdout: true
//                     ).trim()

//                     if (!accuracy || accuracy == "null") {
//                         echo "WARNING: Accuracy not found in metrics.json. Defaulting to 0."
//                         accuracy = "0"
//                     }

//                     env.CURRENT_ACCURACY = accuracy
//                     echo "Current Accuracy: ${env.CURRENT_ACCURACY}"
//                 }
//             }
//         }

//         stage('Compare Accuracy') {
//             steps {
//                 script {
//                     float current = env.CURRENT_ACCURACY.toFloat()
//                     float best = 0.0
//                     boolean improved = false

//                     if (!fileExists(env.BEST_ACCURACY_FILE)) {
//                         echo "No baseline found. First run → promoting model."
//                         improved = true
//                     } else {
//                         best = readFile(env.BEST_ACCURACY_FILE).trim().toFloat()
//                         echo "Best Accuracy: ${best}"

//                         if (current > best) {
//                             improved = true
//                             echo "Model Improved!"
//                         } else {
//                             echo "Model did NOT improve."
//                         }
//                     }

//                     if (improved) {
//                         writeFile file: env.BEST_ACCURACY_FILE, text: "${current}"
//                         env.MODEL_IMPROVED = "true"
//                     } else {
//                         env.MODEL_IMPROVED = "false"
//                     }

//                     echo "MODEL_IMPROVED = ${env.MODEL_IMPROVED}"
//                 }
//             }
//         }

//         stage('Build Docker Image') {
//             when {
//                 expression { env.MODEL_IMPROVED == "false" }  // FIXED: Changed from "false"
//             }
//             steps {
//                 script {
//                     // Unstash model artifacts before building
//                     unstash 'model-artifacts'
                    
//                     // Verify files are present
//                     sh 'ls -la training-artifacts-py3.11/'
                    
//                     docker.withRegistry('', 'dockerhub-creds') {
//                         sh """
//                         docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
//                         docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
//                         """
//                     }
//                 }
//             }
//         }

//         stage('Push Docker Image') {
//             when {
//                 expression { env.MODEL_IMPROVED == "false" }  // FIXED: Changed from "false"
//             }
//             steps {
//                 script {
//                     docker.withRegistry('', 'dockerhub-creds') {
//                         sh """
//                         docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
//                         docker push ${DOCKER_IMAGE}:latest
//                         """
//                     }
//                 }
//             }
//         }
//     }
// }

// lab 7 version

pipeline {
    agent any

    environment {
        VENV_DIR = "venv"
        METRICS_FILE = "training-artifacts-py3.11/metrics.json"
        BEST_ACCURACY_FILE = "best-accuracy"
        DOCKER_IMAGE = "2022bcd0013ashiqfiroz/wine-quality-app-jenkins"
        CURRENT_ACCURACY = "0"
        MODEL_IMPROVED = "false"
        ARTIFACTS_DIR = "training-artifacts-py3.11"

        // Inference testing config
        CONTAINER_NAME = "wine-quality-test-${BUILD_NUMBER}"
        API_PORT = "5000"              // host port
        CONTAINER_PORT = "8002"       // internal app port
        API_HOST = "http://localhost:${API_PORT}"
        HEALTH_ENDPOINT = "/"
        PREDICT_ENDPOINT = "/predict"
        HEALTH_TIMEOUT = "60"
    }

    stages {

        // ─────────────────────────────────────────
        // ORIGINAL TRAINING STAGES
        // ─────────────────────────────────────────

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/2022bcd0013-ashiq-firoz/lab3.git'
            }
        }

        stage('Setup Python Virtual Environment') {
            steps {
                sh '''
                    python3 -m venv $VENV_DIR
                    . $VENV_DIR/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Train Model') {
            steps {
                sh '''
                    . $VENV_DIR/bin/activate

                    mkdir -p output
                    python Script/train.py

                    mkdir -p training-artifacts-py3.11
                    cp -r output/* training-artifacts-py3.11/

                    echo "Contents of training-artifacts-py3.11/:"
                    ls -la training-artifacts-py3.11/
                '''
            }
        }

        stage('Archive Model Artifacts') {
            steps {
                script {
                    sh 'ls -la training-artifacts-py3.11/'
                    archiveArtifacts artifacts: 'training-artifacts-py3.11/**/*', allowEmptyArchive: false
                    stash includes: 'training-artifacts-py3.11/**/*', name: 'model-artifacts'
                    echo "Model artifacts archived successfully"
                }
            }
        }

        stage('Read Accuracy') {
            steps {
                script {
                    if (!fileExists(env.METRICS_FILE)) {
                        echo "WARNING: Metrics file not found. Setting accuracy to 0."
                        env.CURRENT_ACCURACY = "0"
                        return
                    }

                    def accuracy = sh(
                        script: "jq -r '.[-1].accuracy' ${METRICS_FILE} 2>/dev/null || echo 0",
                        returnStdout: true
                    ).trim()

                    if (!accuracy || accuracy == "null") {
                        echo "WARNING: Accuracy not found in metrics.json. Defaulting to 0."
                        accuracy = "0"
                    }

                    env.CURRENT_ACCURACY = accuracy
                    echo "Current Accuracy: ${env.CURRENT_ACCURACY}"
                }
            }
        }

        stage('Compare Accuracy') {
            steps {
                script {
                    float current = env.CURRENT_ACCURACY.toFloat()
                    float best = 0.0
                    boolean improved = false

                    if (!fileExists(env.BEST_ACCURACY_FILE)) {
                        echo "No baseline found. First run → promoting model."
                        improved = true
                    } else {
                        best = readFile(env.BEST_ACCURACY_FILE).trim().toFloat()
                        echo "Best Accuracy: ${best}"
                        if (current > best) {
                            improved = true
                            echo "Model Improved!"
                        } else {
                            echo "Model did NOT improve."
                        }
                    }

                    if (improved) {
                        writeFile file: env.BEST_ACCURACY_FILE, text: "${current}"
                        env.MODEL_IMPROVED = "true"
                    } else {
                        env.MODEL_IMPROVED = "false"
                    }

                    echo "MODEL_IMPROVED = ${env.MODEL_IMPROVED}"
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { env.MODEL_IMPROVED == "true" }
            }
            steps {
                script {
                    unstash 'model-artifacts'
                    sh 'ls -la training-artifacts-py3.11/'

                    docker.withRegistry('', 'dockerhub-creds') {
                        sh """
                        docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                        docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Push Docker Image') {
            when {
                expression { env.MODEL_IMPROVED == "true" }
            }
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        sh """
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 1 : Pull Inference Image
        // ─────────────────────────────────────────

        stage('Stage 1: Pull Image') {
            steps {
                script {
                    echo "============================================"
                    echo "STAGE 1: Pulling inference image from Docker Hub"
                    echo "============================================"

                    sh "docker pull ${DOCKER_IMAGE}:latest"

                    // Verify the image is present locally
                    def imageCheck = sh(
                        script: "docker image inspect ${DOCKER_IMAGE}:latest --format '{{.Id}}' 2>/dev/null || echo 'NOT_FOUND'",
                        returnStdout: true
                    ).trim()

                    if (imageCheck == 'NOT_FOUND' || imageCheck == '') {
                        error("Image pull verification FAILED: image not found locally after pull.")
                    }

                    echo "✔ Image pulled and verified successfully. Image ID: ${imageCheck}"
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 2 : Run Container
        // ─────────────────────────────────────────

        stage('Stage 2: Run Container') {
            steps {
                script {
                    echo "============================================"
                    echo "STAGE 2: Starting inference container"
                    echo "Container name : ${CONTAINER_NAME}"
                    echo "Exposed port   : ${API_PORT}"
                    echo "============================================"

                    

                    // Remove any leftover container with the same name
                    sh "docker rm -f ${CONTAINER_NAME} 2>/dev/null || true"

                    sh """
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        ${DOCKER_IMAGE}:latest
                    """

                    def containerId = sh(
                        script: "docker inspect -f '{{.Id}}' ${CONTAINER_NAME}",
                        returnStdout: true
                    ).trim()

                    echo "✔ Container started. ID: ${containerId}"
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 3 : Wait for Service Readiness
        // ─────────────────────────────────────────

        stage('Stage 3: Wait for Service Readiness') {
            steps {
                script {
                    echo "============================================"
                    echo "STAGE 3: Waiting for API to become ready"
                    echo "Health endpoint : ${API_HOST}${HEALTH_ENDPOINT}"
                    echo "Timeout         : ${HEALTH_TIMEOUT}s"
                    echo "============================================"

                    def ready = false
                    def elapsed = 0
                    def interval = 5

                    while (elapsed < HEALTH_TIMEOUT.toInteger()) {
                        def statusCode = sh(
                            script: "curl -s -o /dev/null -w '%{http_code}' ${API_HOST}${HEALTH_ENDPOINT} 2>/dev/null || echo '000'",
                            returnStdout: true
                        ).trim()

                        echo "[${elapsed}s] Health check → HTTP ${statusCode}"

                        if (statusCode == '200') {
                            ready = true
                            break
                        }

                        sleep interval
                        elapsed += interval
                    }

                    if (!ready) {
                        // Dump container logs before failing for easier debugging
                        sh "docker logs ${CONTAINER_NAME} || true"
                        error("✘ Service did NOT become ready within ${HEALTH_TIMEOUT}s. Pipeline FAILED.")
                    }

                    echo "✔ Service is ready and responding."
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 4 : Send Valid Inference Request
        // ─────────────────────────────────────────

        stage('Stage 4: Send Valid Inference Request') {
            steps {
                script {
                    echo "============================================"
                    echo "STAGE 4: Sending valid inference request"
                    echo "============================================"

                    // Valid wine-quality feature payload (11 features)
                    def validPayload = '''{
                        "fixed acidity": 7.4,
                        "volatile acidity": 0.70,
                        "citric acid": 0.00,
                        "residual sugar": 1.9,
                        "chlorides": 0.076,
                        "free sulfur dioxide": 11.0,
                        "total sulfur dioxide": 34.0,
                        "density": 0.9978,
                        "pH": 3.51,
                        "sulphates": 0.56,
                        "alcohol": 9.4
                    }'''

                    def response = sh(
                        script: """
                            curl -s -w '\\nHTTP_STATUS:%{http_code}' \
                                -X POST \
                                -H 'Content-Type: application/json' \
                                -d '${validPayload}' \
                                ${API_HOST}${PREDICT_ENDPOINT}
                        """,
                        returnStdout: true
                    ).trim()

                    echo "--- Raw API Response ---"
                    echo response
                    echo "------------------------"

                    // Split body and status code
                    def parts     = response.split('HTTP_STATUS:')
                    def body      = parts[0].trim()
                    def httpCode  = parts.length > 1 ? parts[1].trim() : '000'

                    echo "HTTP Status Code : ${httpCode}"
                    echo "Response Body    : ${body}"

                    // Validation 1 – HTTP status must be 2xx
                    if (!(httpCode ==~ /2\d\d/)) {
                        error("✘ Valid request FAILED: Expected 2xx HTTP status, got ${httpCode}.")
                    }
                    echo "✔ HTTP status is successful (${httpCode})."

                    // Validation 2 – 'prediction' field must exist
                    def hasPrediction = sh(
                        script: "echo '${body}' | jq 'has(\"prediction\")' 2>/dev/null || echo 'false'",
                        returnStdout: true
                    ).trim()

                    if (hasPrediction != 'true') {
                        error("✘ Valid request FAILED: 'prediction' field missing in response body.")
                    }
                    echo "✔ 'prediction' field exists in response."

                    // Validation 3 – prediction value must be numeric
                    def predValue = sh(
                        script: "echo '${body}' | jq -r '.prediction' 2>/dev/null || echo 'null'",
                        returnStdout: true
                    ).trim()

                    echo "Prediction value : ${predValue}"

                    def isNumeric = sh(
                        script: "echo '${body}' | jq '.prediction | type == \"number\"' 2>/dev/null || echo 'false'",
                        returnStdout: true
                    ).trim()

                    if (isNumeric != 'true') {
                        error("✘ Valid request FAILED: 'prediction' value '${predValue}' is not numeric.")
                    }

                    echo "✔ Prediction value is numeric (${predValue}). All validations PASSED."
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 5 : Send Invalid Request
        // ─────────────────────────────────────────

        stage('Stage 5: Send Invalid Request') {
            steps {
                script {
                    echo "============================================"
                    echo "STAGE 5: Sending invalid / malformed request"
                    echo "============================================"

                    // Malformed payload – missing required fields, wrong types
                    def invalidPayload = '''{
                        "fixed acidity": "not-a-number",
                        "volatile acidity": null
                    }'''

                    def response = sh(
                        script: """
                            curl -s -w '\\nHTTP_STATUS:%{http_code}' \
                                -X POST \
                                -H 'Content-Type: application/json' \
                                -d '${invalidPayload}' \
                                ${API_HOST}${PREDICT_ENDPOINT}
                        """,
                        returnStdout: true
                    ).trim()

                    echo "--- Raw API Response (Invalid Input) ---"
                    echo response
                    echo "----------------------------------------"

                    def parts    = response.split('HTTP_STATUS:')
                    def body     = parts[0].trim()
                    def httpCode = parts.length > 1 ? parts[1].trim() : '000'

                    echo "HTTP Status Code : ${httpCode}"
                    echo "Response Body    : ${body}"

                    // Validation 1 – must NOT return 2xx (expected 4xx or 5xx)
                    if (httpCode ==~ /2\d\d/) {
                        error("✘ Invalid request FAILED: API returned ${httpCode} for bad input; expected 4xx/5xx error response.")
                    }
                    echo "✔ API correctly returned error HTTP status (${httpCode}) for invalid input."

                    // Validation 2 – error message must be present and meaningful
                    def hasError = sh(
                        script: "echo '${body}' | jq 'has(\"error\") or has(\"message\") or has(\"detail\")' 2>/dev/null || echo 'false'",
                        returnStdout: true
                    ).trim()

                    if (hasError != 'true') {
                        error("✘ Invalid request FAILED: Response contains no 'error', 'message', or 'detail' field. Error messages must be meaningful.")
                    }

                    // Extract whichever error field is present
                    def errorMsg = sh(
                        script: """
                            echo '${body}' | jq -r '
                                if has("error") then .error
                                elif has("message") then .message
                                elif has("detail") then .detail
                                else "N/A" end
                            ' 2>/dev/null || echo 'N/A'
                        """,
                        returnStdout: true
                    ).trim()

                    if (!errorMsg || errorMsg == 'N/A' || errorMsg.length() < 5) {
                        error("✘ Invalid request FAILED: Error message '${errorMsg}' is not meaningful (too short or empty).")
                    }

                    echo "✔ Meaningful error message received: \"${errorMsg}\""
                    echo "✔ Invalid request handling PASSED."
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 6 : Stop Container
        // ─────────────────────────────────────────

        stage('Stage 6: Stop Container') {
            steps {
                script {
                    echo "============================================"
                    echo "STAGE 6: Stopping and removing test container"
                    echo "============================================"

                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm   ${CONTAINER_NAME} || true"

                    // Verify no leftover running container
                    def running = sh(
                        script: "docker ps --filter name=${CONTAINER_NAME} --format '{{.Names}}' 2>/dev/null || echo ''",
                        returnStdout: true
                    ).trim()

                    if (running) {
                        error("✘ Container '${CONTAINER_NAME}' is still running after stop/remove attempt.")
                    }

                    echo "✔ Container stopped and removed. No leftover running containers."
                }
            }
        }
    }

    // ─────────────────────────────────────────
    // STAGE 7 : Pipeline Result (post block)
    // ─────────────────────────────────────────

    post {
        success {
            echo "============================================"
            echo "STAGE 7: PIPELINE RESULT → ✔ SUCCESS"
            echo "All inference validation tests PASSED."
            echo "============================================"
        }
        failure {
            script {
                echo "============================================"
                echo "STAGE 7: PIPELINE RESULT → ✘ FAILED"
                echo "One or more validation steps FAILED."
                echo "Attempting emergency container cleanup..."
                echo "============================================"

                // Best-effort cleanup so no stale container is left behind
                sh "docker stop ${CONTAINER_NAME} 2>/dev/null || true"
                sh "docker rm   ${CONTAINER_NAME} 2>/dev/null || true"
            }
        }
        always {
            echo "Pipeline finished. Check console output above for per-stage pass/fail status."
        }
    }
}