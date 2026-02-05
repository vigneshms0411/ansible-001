pipeline {
    agent any
    
    environment {
        ANSIBLE_REPO = 'https://github.com/vigneshms0411/ansible-001.git'
        WEBSITE_REPO = 'https://github.com/vigneshms0411/jenkins.git'
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout Ansible Code') {
            steps {
                script {
                    dir('ansible-code') {
                        git branch: 'main',
                            url: "${ANSIBLE_REPO}",
                            credentialsId: 'github-credentials'
                    }
                }
            }
        }
        
        stage('Checkout Website Code') {
            steps {
                script {
                    dir('ansible-code/web-src') {
                        git branch: 'main',
                            url: "${WEBSITE_REPO}",
                            credentialsId: 'github-credentials'
                    }
                }
            }
        }
        
        stage('Verify Files') {
            steps {
                script {
                    dir('ansible-code') {
                        sh '''
                            echo "=== Ansible Repository Structure ==="
                            ls -la
                            echo ""
                            echo "=== Inventory File Content ==="
                            cat inventory/hosts.ini
                            echo ""
                            echo "=== Website Files ==="
                            ls -la web-src/
                        '''
                    }
                }
            }
        }
        
        stage('Install Apache & Deploy Website') {
            steps {
                script {
                    dir('ansible-code') {
                        ansiblePlaybook(
                            playbook: 'deploy.yml',
                            inventory: 'inventory/hosts.ini',
                            credentialsId: 'aws-ssh-key',
                            colorized: true,
                            disableHostKeyChecking: true,
                            extras: '-vvv'
                        )
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    echo '========================================='
                    echo '✅ Deployment Complete!'
                    echo '========================================='
                    echo ''
                    echo 'Access your website at:'
                    echo '  → http://172.31.39.174'
                    echo '  → http://172.31.40.161'
                    echo ''
                }
            }
        }
    }
    
    post {
        success {
            echo ''
            echo '╔════════════════════════════════════════╗'
            echo '║   ✅ DEPLOYMENT SUCCESSFUL!           ║'
            echo '╚════════════════════════════════════════╝'
            echo ''
        }
        failure {
            echo ''
            echo '╔════════════════════════════════════════╗'
            echo '║   ❌ DEPLOYMENT FAILED!               ║'
            echo '╚════════════════════════════════════════╝'
            echo ''
        }
        always {
            cleanWs()
        }
    }
}
