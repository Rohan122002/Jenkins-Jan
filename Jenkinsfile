pipeline {
    agent any

    environment {
        EC2_USER   = "ec2-user"
        EC2_IP     = "34.224.25.248"
        SSH_KEY    = "/var/lib/jenkins/.ssh/performance-ci-new.pem"
        JMETER     = "/opt/apache-jmeter-5.6.3/bin/jmeter"
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
                    rm -rf ${REMOTE_DIR}/performance-ci
                    mkdir -p ${REMOTE_DIR}/performance-ci
                    mkdir -p ${REMOTE_DIR}/results/{sanity,load}/report
                '
                """

                sh """
                scp -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} -r Jmeter \
                ${EC2_USER}@${EC2_IP}:${REMOTE_DIR}/performance-ci/
                """
            }
        }

        /* ================= SANITY ================= */

        stage('Sanity Test') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                    rm -f ${REMOTE_DIR}/results/sanity.jtl
                    rm -rf ${REMOTE_DIR}/results/sanity/report
                    mkdir -p ${REMOTE_DIR}/results/sanity/report

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

        /* ================= LOAD ================= */

        stage('Load Test') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                    rm -f ${REMOTE_DIR}/results/load.jtl
                    rm -rf ${REMOTE_DIR}/results/load/report
                    mkdir -p ${REMOTE_DIR}/results/load/report

                    ${JMETER} -n \
                    -t ${REMOTE_DIR}/performance-ci/Jmeter/Load_test.jmx \
                    -l ${REMOTE_DIR}/results/load.jtl \
                    -e -o ${REMOTE_DIR}/results/load/report
                '
                """
            }
        }

        stage('Validate Load') {
            steps {
                sh """
                ssh -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                    if grep -q ",false," ${REMOTE_DIR}/results/load.jtl; then
                        echo "❌ Load test failed"
                        exit 1
                    else
                        echo "✅ Load test passed"
                    fi
                '
                """
            }
        }
    }

    post {
        success {
            echo "✅ Sanity + Load pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed due to performance issues"
        }
        always {
            echo "🏁 Performance execution completed"
        }
    }
}

