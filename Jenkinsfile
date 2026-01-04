pipeline {
    agent any

    environment {
        EC2_USER = "ec2-user"
        EC2_IP   = "16.16.215.173"
        SSH_KEY  = "/home/rohan/Public/Projects/JenkinsProject/performance-ci.pem"
        JMETER   = "/opt/jmeter/bin/jmeter"
        REMOTE_DIR = "/home/ec2-user/jmeter-tests"
        REPO_NAME = "Jenkins-Jan"
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
                ssh -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                mkdir -p ${REMOTE_DIR}/results/{sanity,load,stress,soak}/report
                rm -rf ${REMOTE_DIR}/${REPO_NAME}
                '
                """

                sh """
                scp -i ${SSH_KEY} -r . ${EC2_USER}@${EC2_IP}:${REMOTE_DIR}/
                """
            }
        }

        stage('Sanity Test') {
            steps {
                sh """
                ssh -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                ${JMETER} -n \
                -t ${REMOTE_DIR}/${REPO_NAME}/Jmeter/Sanity.jmx \
                -l ${REMOTE_DIR}/results/sanity.jtl \
                -e -o ${REMOTE_DIR}/results/sanity/report
                '
                """
            }
        }

        stage('Validate Sanity') {
            steps {
                sh """
                ssh -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                if grep -q ",false," ${REMOTE_DIR}/results/sanity.jtl; then
                    echo "Sanity test failed"
                    exit 1
                fi
                '
                """
            }
        }

        stage('Load Test') {
            steps {
                sh """
                ssh -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                ${JMETER} -n \
                -t ${REMOTE_DIR}/${REPO_NAME}/Jmeter/Load_Test.jmx \
                -l ${REMOTE_DIR}/results/load.jtl \
                -e -o ${REMOTE_DIR}/results/load/report
                '
                """
            }
        }

        stage('Stress Test') {
            steps {
                sh """
                ssh -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                ${JMETER} -n \
                -t ${REMOTE_DIR}/${REPO_NAME}/Jmeter/Stress_Test.jmx \
                -l ${REMOTE_DIR}/results/stress.jtl \
                -e -o ${REMOTE_DIR}/results/stress/report
                '
                """
            }
        }

        stage('Soak Test') {
            steps {
                sh """
                ssh -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
                ${JMETER} -n \
                -t ${REMOTE_DIR}/${REPO_NAME}/Jmeter/Soak_Test.jmx \
                -l ${REMOTE_DIR}/results/soak.jtl \
                -e -o ${REMOTE_DIR}/results/soak/report
                '
                """
            }
        }
    }

    post {
        always {
            echo "Performance pipeline execution completed"
        }
        failure {
            echo "Pipeline failed due to performance test failure"
        }
        success {
            echo "All performance tests executed successfully"
        }
    }
}

