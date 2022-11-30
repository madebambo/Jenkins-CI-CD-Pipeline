def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    
    
    tools {
        maven 'localMaven'
        jdk 'localJdk'
    }
    
    environment {
        WORKSPACE = "${env.WORKSPACE}"
        
        
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/cvamsikrishna11/Jenkins-CI-CD-Pipeline-Project-V1.git'
                
            }
        }
        
        
        stage('Build') {
            steps {
                sh "mvn clean package"
                
            }
            
            post {
                success {
                   archiveArtifacts artifacts: '**/*.war', followSymlinks: false 
                }
            }
            
        }
        
        
    stage('Unit Test'){
        steps {
            sh 'mvn test'
        }
    }
    stage('Integration Test'){
        steps {
          sh 'mvn verify -DskipUnitTests'
        }
    }
    stage ('Checkstyle Code Analysis'){
        steps {
            sh 'mvn checkstyle:checkstyle'
        }
        post {
            success {
                echo 'Generated Analysis Result'
            }
        }
    }
    
    stage ('SonarQube Scan'){
        steps {
            
            withSonarQubeEnv('SonarQube') {
            sh '''
            mvn sonar:sonar \
          -Dsonar.projectKey=JavaWebApp \
          -Dsonar.host.url=http://172.31.27.36:9000 \
          -Dsonar.login=49c9b18dbd60133d4772fbd100ed88f777d7d041

            '''
            }
                
            }
    }
    
    
    stage("Quality Gate"){
        
      steps{
           
       waitForQualityGate abortPipeline: true
           
      }
        
      }
      
      
      
    stage("Upload artifact to Nexus"){
        
      steps{
           
        sh 'mvn clean deploy -DskipTests'
           
      }
        
      }
      
      
      
      
      stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh 'ls'
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
     
     
    stage('Deploy to STAGE env') {
      environment {
        HOSTS = "stage"
      }
      steps {
        sh 'ls'
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
     
     
    
     stage('Approval') {
      steps {
        input('Do you want to proceed?')
      }
    } 
     
     
     
    stage('Deploy to PROD env') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh 'ls'
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
      
   
    }
    
    
    post {
    always {
        echo 'Slack Notifications.'
        slackSend channel: '#team-devops', //update and provide your channel name
        color: COLOR_MAP[currentBuild.currentResult],
        message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL} - vamsi"
    }
  }
    
    
    
}
