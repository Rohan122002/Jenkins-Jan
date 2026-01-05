pipeline {
    agent any

    environment {
        EC2_USER   = "ec2-user"
        EC2_IP     = "54.145.38.103"
        SSH_KEY    = "/var/lib/jenkins/.ssh/performance-ci-new.pem"
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
                    rm -rf ${REMOTE_DIR}/performance-ci
                    mkdir -p ${REMOTE_DIR}/performance-ci
                    mkdir -p ${REMOTE_DIR}/results/sanity/report
                '
                """

                sh """
                scp -o StrictHostKeyChecking=no \
                -i ${SSH_KEY} -r Jmeter \
                ${EC2_USER}@${EC2_IP}:${REMOTE_DIR}/performance-ci/
                """
            }
        }

        stage('Sanity Test') {
    steps {
        sh """
        ssh -o StrictHostKeyChecking=no \
        -i ${SSH_KEY} ${EC2_USER}@${EC2_IP} '
            # Clean previous results
            rm -f ${REMOTE_DIR}/results/sanity.jtl
            rm -rf ${REMOTE_DIR}/results/sanity/report
            mkdir -p ${REMOTE_DIR}/results/sanity/report

            # Run JMeter sanity
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
                    echo "✅ Sanity test passed1111"
                fi
                '
                """
            }
        }
    }

    post {
        success {
            echo "✅ Sanity pipeline completed successfully"
        }
        failure {
            echo "❌ Sanity pipeline failed"
        }
        always {
            echo "🏁 Sanity execution completed"
        }
    }
}

