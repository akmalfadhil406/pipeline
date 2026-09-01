pipeline {
    agent any

    parameters {
        string(name: 'TARGET_HOST', defaultValue: '', description: 'RHEL server IP or hostname to configure')
        string(name: 'PACKAGES', defaultValue: 'httpd,git,vim', description: 'Comma-separated list of packages to install')
        string(name: 'ANSIBLE_USER', defaultValue: 'ansible', description: 'SSH user Ansible connects as on the target')
    }

    environment {
        // ID of an "SSH Username with private key" credential in Jenkins,
        // set up for the ANSIBLE_USER account on the RHEL target.
        SSH_CRED_ID = 'rhel-ansible-ssh-key'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Input') {
            steps {
                script {
                    if (!params.TARGET_HOST?.trim()) {
                        error("TARGET_HOST parameter is required.")
                    }
                }
            }
        }

        stage('Check Ansible Installed') {
            steps {
                sh 'ansible --version'
            }
        }

        stage('Generate Inventory') {
            steps {
                writeFile file: 'inventory.ini', text: """[rhel_targets]
${params.TARGET_HOST} ansible_user=${params.ANSIBLE_USER}
"""
            }
        }

        stage('Run Playbook') {
            steps {
                sshagent(credentials: [env.SSH_CRED_ID]) {
                    sh """
                        ansible-playbook -i inventory.ini install_packages.yml \
                          --extra-vars "packages='${params.PACKAGES}'" \
                          -e ansible_ssh_common_args='-o StrictHostKeyChecking=no'
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Packages [${params.PACKAGES}] installed successfully on ${params.TARGET_HOST}."
        }
        failure {
            echo "Pipeline failed. Check the Ansible output above for details."
        }
        always {
            sh 'rm -f inventory.ini'
        }
    }
}
