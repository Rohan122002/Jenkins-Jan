pipeline {
    agent any

    environment {
        EC2_USER   = "ec2-user"
        EC2_IP     = "54.145.38.103"
        SSH_KEY    = "/var/lib/jenkins/.ssh/performance-ci.pem"
        JMETER     = "/opt/jmeter/bin/jmeter"
        REMOTE_DIR = "/home/ec2-user/jmeter-tests"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rohan122002/Jenkins-Jan.git'
            }
        }

        stage('Prepare EC2') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                    mkdir -p ${REMOTE_DIR}/results/{sanity,load,stress,soak}/report
                    rm -rf ${REMOTE_DIR}/performance-ci
                '
                """

                sh """
                scp -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} -r ${WORKSPACE} \
                ${EC2_USER}@${EC2_IP}:${REMOTE_DIR}/performance-ci
                """
            }
        }

        stage('Sanity Test') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                ${JMETER} -n \
                -t ${REMOTE_DIR}/performance-ci/Jmeter/Sanity.jmx \
                -l ${REMOTE_DIR}/results/sanity.jtl \
                -e -o ${REMOTE_DIR}/results/sanity/report
                '
                """
            }
        }

        stage('Validate Sanity') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                if grep -q ",false," ${REMOTE_DIR}/results/sanity.jtl; then
                    echo "❌ Sanity test failed"
                    exit 1
                else
                    echo "✅ Sanity test passed"
                fi
                '
                """
            }
        }

        stage('Load Test') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                ${JMETER} -n \
                -t ${REMOTE_DIR}/performance-ci/Jmeter/Load_test.jmx \
                -l ${REMOTE_DIR}/results/load.jtl \
                -e -o ${REMOTE_DIR}/results/load/report
                '
                """
            }
        }

        stage('Stress Test') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                ${JMETER} -n \
                -t ${REMOTE_DIR}/performance-ci/Jmeter/Stress_Test.jmx \
                -l ${REMOTE_DIR}/results/stress.jtl \
                -e -o ${REMOTE_DIR}/results/stress/report
                '
                """
            }
        }

        stage('Soak Test') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                ${JMETER} -n \
                -t ${REMOTE_DIR}/performance-ci/Jmeter/Soak_Test.jmx \
                -l ${REMOTE_DIR}/results/soak.jtl \
                -e -o ${REMOTE_DIR}/results/soak/report
                '
                """
            }
        }
    }

    post {
        success {
            echo "✅ All performance tests executed successfully"
        }
        failure {
            echo "❌ Pipeline failed due to performance test failure"
        }
        always {
            echo "🏁 Performance pipeline execution completed"
        }
    }
}

