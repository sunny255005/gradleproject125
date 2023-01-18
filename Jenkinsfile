import groovy.json.JsonSlurper

pipeline{
    environment {
        PROD_BRANCH = 'master'
        STAGING_BRANCH = 'staging'
        user_env_input = 'Development'
        is_unit_test_continue='No'
        is_sonarqube='No'
        GIT_REPO_NAME = GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
        is_ready='Yes'
    }
    
    

    agent any
    tools { 
       gradle'gradle7' 
     
       
    } 
   
    stages {
        stage('Which environment to build?') {
            steps {
            sh 'echo which environment to build'      
                
            }
        }
        stage('Confirm') {
            steps {
                script {
             
                        input("Do you want to proceed building in ${user_env_input} environment?")
                    }
                }
            
        }
        stage('Docker Build') {
            steps {
                
                echo 'docker build..'
            }
        }
        

        stage('Confirm for unit tests') {
            steps {
                 
               script{
                   def is_unit_test_continue_parameter = input(id: 'is_unit_test_continue', message: 'Do you want to go for unit tests?',
                    parameters: [[$class: 'ChoiceParameterDefinition', defaultValue: 'No',
                        description:'Unit Test choices', name:'invalidate_cf_params', choices: 'Yes\nNo']
                    ])
                    
                   
                    is_unit_test_continue=is_unit_test_continue_parameter


                  

               }}}
        stage('Unit Tests & Jacoco Reports') {
            when {
         expression { is_unit_test_continue == "Yes" }
     }
     steps {
         echo "Hello,unit_test continue...!"
            script {
                sh './gradlew test'
                echo 'testing in progess...'
                 jacoco()
             //junit ''
                junit testResults: '**/test-results/test/*.xml', skipPublishingChecks: true
            }
        }
        }
        
 
//  stage('Code Coverage with jaccoco'){
//             steps{   
//         jacoco()
//             }
//         }
        
        stage('Confirm for Sonarqube Check') {
            steps {
                 
               script{

                

                def is_sonarqube_parameter = input(id: 'is_sonarqube', message: 'Do you want to continue with Sonarqube?',
                    parameters: [[$class: 'ChoiceParameterDefinition', defaultValue: 'No',
                        description:'Sonarqube choices', name:'invalidate_cf_params', choices: 'Yes\nNo']
                    ])
                    
                   
                   is_sonarqube=is_sonarqube_parameter
                   
//    def response = httpRequest 'http://44.227.115.141:9000'
//         println("Status: "+response.status)
//         println("Content: "+response.content)
        
                  
       

               }}}


                stage('Sonarqube Integeration') {
            when {
         expression { is_sonarqube == "Yes"}
     }
     steps {
        //  hello=sh'curl --location --request GET -w "%{http_code}" http://44.227.115.141:9000/'
        //     echo hello

       
         echo "Hello,sonarqube continue...!"
            script {

                  
timeout(time: 1, unit: 'MINUTES') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = httpRequest 'http://44.227.115.141:9000' // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
        is_ready='No'
      error "Sonarqube Server may be not running,so Going to Next Stage"
    }
  
                   }
                //def url = sh(returnStdout: true, script: 'git config remote.origin.url').trim()
                //sh 'echo url'
                
                if(is_ready=='Yes'){
                 withSonarQubeEnv(installationName: 'sonarqube-server', credentialsId: 'sonarqube-secret-token') {
                    

                     sh './gradlew sonarqube \
                     -Dsonar.projectName=${GIT_REPO_NAME} \
  -Dsonar.host.url=http://localhost:9001 \
      -Dsonar.projectKey=test  \
'
                     

                 }
                }

  timeout(time: 1, unit: 'MINUTES') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
                  
                
                        
                        
                    
            }
          
        }
                 
                  

 
  

           
                }
       

    
            
            
stage("finished")
{
    steps{
        sh'echo finsihed'
    }
}

        }
}

        
    


