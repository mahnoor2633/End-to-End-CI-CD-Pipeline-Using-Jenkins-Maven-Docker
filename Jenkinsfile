pipeline {
  agent any

  environment {
    DOCKER_SERVER = "3.27.247.137"
    DOCKER_USER = "ubuntu"              // or ubuntu, depending on your OS
    REMOTE_DIR  = "/opt/Docker"
    APP_NAME    = "webapp"
    HOST_PORT   = "8086"
    CONT_PORT   = "8080"
    WAR_PATH    = "webapp/target/webapp.war"
  }

  stages {
    stage("Checkout") {
      steps { checkout scm }
    }

  stage('Build WAR') {
      steps {
        script {
          def mvnHome = tool 'Maven'   // must match the name in Global Tool Configuration
          sh """
            ${mvnHome}/bin/mvn -B -DskipTests clean package
          """
        }
      }
    }

    stage("Ship WAR to Docker Host") {
      steps {
        // Use SSH key auth from Jenkins credentials (recommended)
        sshagent(credentials: ['DOCKER_HOST_SSH_KEY_ID']) {
          sh """
            scp -o StrictHostKeyChecking=no ${WAR_PATH} ${DOCKER_USER}@${DOCKER_SERVER}:${REMOTE_DIR}/webapp.war
          """
        }
      }
    }

    stage("Deploy on Docker Host") {
      steps {
        sshagent(credentials: ['DOCKER_HOST_SSH_KEY_ID']) {
          sh '''
          ssh -o StrictHostKeyChecking=no ${DOCKER_USER}@${DOCKER_SERVER} DOCKER_HOST="${DOCKER_HOST}" APP_NAME="${APP_NAME}" BUILD_NUMBER="${BUILD_NUMBER}" REMOTE_DIR="${REMOTE_DIR}" HOST_PORT="${HOST_PORT}" CONT_PORT="${CONT_PORT}" bash -se << 'EOF'
          set -e
          cd ${REMOTE_DIR}
          mkdir -p war_hist
          TS=$(date +%Y%m%d_%H%M%S)
    
                  # archive WAR
          cp -f webapp.war "war_hist/webapp-${BUILD_NUMBER}-${TS}.war"
    
                  # build versioned image + latest
          echo ${APP_NAME}
          docker build -t ${APP_NAME}:${BUILD_NUMBER} -t ${APP_NAME}:latest .
    
                  # restart container to new build
          docker rm -f ${APP_NAME} 2>/dev/null || true
          docker run -d --restart=always --name ${APP_NAME} -p ${HOST_PORT}:${CONT_PORT} ${APP_NAME}:${BUILD_NUMBER}
    
                  # Find previous numeric tag (rollback candidate)
          PREV=$(docker images ${APP_NAME} --format '{{.Tag}}' | grep -E '^[0-9]+$' | grep -v "^${BUILD_NUMBER}$" | sort -nr | head -n 1 || true)
                  
                  # Health check loop
          URL="http://${DOCKER_HOST}:${HOST_PORT}/${APP_NAME}/"   # change path if needed
          echo "URL=${URL}"
          MAX=20
          SLEEP=3
          ok=0
                  
          for i in $(seq 1 ${MAX}); do
            code=$(curl -s -o /dev/null -w "%{http_code}" "${URL}" || true)
            if [ "${code}" = "200" ] || [ "${code}" = "302" ]; then
              ok=1
              echo "Healthy (HTTP ${code})"
              break
            fi
            echo "Not healthy yet (HTTP ${code}) attempt ${i}/${MAX}"
            sleep ${SLEEP}
          done
                  
          if [ "${ok}" -ne 1 ]; then
            echo "Health check failed. Rolling back..."
            docker rm -f ${APP_NAME} 2>/dev/null || true
            if [ -n "${PREV}" ]; then
              docker run -d --restart=always --name ${APP_NAME} -p ${HOST_PORT}:${CONT_PORT} ${APP_NAME}:${PREV}
            else
              echo "No previous image available for rollback."
              exit 1
            fi
          fi
    
          echo "APP_NAME=${APP_NAME}"
          echo "BUILD_NUMBER=${BUILD_NUMBER}"
          echo "REMOTE_DIR=${REMOTE_DIR}" EOF
          '''
        }
      }
    }
  }
}
