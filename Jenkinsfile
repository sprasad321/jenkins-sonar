pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/sprasad321/jenkins-sonar.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('LocalSonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Deploy WAR to Tomcat') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'vm1-key', keyFileVariable: 'SSH_KEY_FILE', usernameVariable: 'SSH_USER'),
                    string(credentialsId: 'vault-password-id', variable: 'VAULT_PASSWORD')
                ]) {
                    sh '''
                        # Fix permissions for SSH key
                        chmod 600 $SSH_KEY_FILE

                        # Write vault password
                        echo $VAULT_PASSWORD > ansible/vault/.vault_pass.txt

                        # Run Ansible playbook
                        ansible-playbook \
                        -i ansible/inventories/production \
                        ansible/playbooks/deploy-tomcat.yml \
                        --private-key $SSH_KEY_FILE \
                        -u $SSH_USER \
                        --vault-password-file ansible/vault/.vault_pass.txt
                    '''
                }
            }
        }
    }
}
