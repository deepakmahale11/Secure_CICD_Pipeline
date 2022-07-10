//pipeline code
pipeline {
    agent any
    stages {
        stage('Networking Configuration') {
            steps {
                sh 'docker network ls'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'cd $WORKSPACE'
                sh 'rm -rf CDAC-Project'
                git branch: "master",
                    url: "https://github.com/RaziAbbas1/Devsecops"
                sh 'ls'
            }
        }
        stage('Pre-Build Tests') {
            parallel {
                stage('Git Repository Scanner') {
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'trufflehog https://github.com/RaziAbbas1/Devsecops --json | jq "{branch:.branch, commitHash:.commitHash, path:.path, stringsFound:.stringsFound}" > trufflehog_report.json || true'
                        sh 'cat trufflehog_report.json'
                        sh 'echo "Scanning Repositories.....done"'
                        archiveArtifacts artifacts: 'trufflehog_report.json', onlyIfSuccessful: true
                        //emailext attachLog: true, attachmentsPattern: 'trufflehog_*', 
                        //body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Thankyou,\n CDAC-Project Group-11", 
                        //subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME} - success", mimeType: 'text/html', to: "raziabbasrizvi75@gmail.com"
                    }
                }
                stage('Image Security') {
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'dockle --input ~/docker_img_backup/mytomcat.tar -f json -o mytomcat_report.json'
                        sh 'dockle --input ~/docker_img_backup/pgadmin4.tar -f json -o pgadmin4_report.json'
                        sh 'dockle --input ~/docker_img_backup/postgres11.tar -f json -o postgres11_report.json'
                        //sh 'dockle --input ~/docker_img_backup/zap2docker-stable.tar -f json -o zap2docker-stable_report.json'
                        sh 'dockle --input ~/docker_img_backup/sonarqube.tar -f json -o sonarqube_report.json'
                        archiveArtifacts artifacts: '*.json', onlyIfSuccessful: true
                        //emailext attachLog: true, attachmentsPattern: '*.json', 
                        //body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-11",
                        //subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "raziabbasrizvi75@gmail.com"
                    }
                }
            }
        }
        stage('Build Stage') {
            steps {
                sh 'mvn clean'
                sh 'mvn compile'
                sh 'mvn install'
                sh 'mvn package'
            }
        }
         stage('Initializing Docker') {
            parallel {
                stage('Build Docker Images') {
                    steps {
                        sh 'docker build -t mytomcat .'
                        sh 'docker-compose up -d'
                    }
                }
                stage('Deploying Containers') {
                    steps {
                        sh 'docker start mytomcat'
                        //sh 'docker stop pgadmin_container'
                        //sh 'docker stop postgres_container'
                        //sh 'docker stop login'
                        //sh 'docker start pgadmin_container'
                        //sh 'docker start postgres_container'
                        //sh 'docker start login'
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                sh 'mvn sonar:sonar -Dsonar.projectKey=group11 -Dsonar.host.url=http://cdac.project.com:4444 -Dsonar.login=1fa472016347b9ec6ac4028f3cbd6f082b4bd735 || true'
            }
        }
       
        stage('DAST') {
            steps {
                sh 'docker rm dast_baseline'
                sh 'docker rm dast_full'
                sh 'docker run -u root --name dast_full -v $(pwd):/zap/wrk/:Z -t owasp/zap2docker-stable zap-full-scan.py -t http://cdac.project.com/LoginWebApp/ -r full_scan.html || true'
                sh 'docker run -u root --name dast_baseline -v $(pwd):/zap/wrk/:Z -t owasp/zap2docker-stable zap-baseline.py -t http://cdac.project.com/LoginWebApp/ --autooff -r baseline_scan.html || true'
                archiveArtifacts artifacts: 'full_scan.html', onlyIfSuccessful: true
                archiveArtifacts artifacts: 'baseline_scan.html', onlyIfSuccessful: true
                emailext attachLog: false, attachmentsPattern: '*scan.html', 
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-11",
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "raziabbasrizvi75@gmail.com"
            }
        }
    }
}
