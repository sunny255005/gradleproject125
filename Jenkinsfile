String determineRepoName() {
    return scm.getUserRemoteConfigs()[0].getUrl().tokenize('/')[3].split("\\.")[0]
}

pipeline{
    environment {
        PROD_BRANCH = 'master'
        STAGING_BRANCH = 'staging'
        user_env_input = 'Development'
        is_unit_test_continue='No'
        is_sonarqube='No'
        //GIT_REPO_NAME = GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
        GIT_REPO_NAME=determineRepoName()
        
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
                script{
                cleanWs()
                def gitRemoteOriginUrl = scm.getUserRemoteConfigs()[0].getUrl()
                        echo 'The remote URL is ' + gitRemoteOriginUrl
                echo 'docker build..${BRANCH_NAME}'
                    echo "${GIT_REPO_NAME}"
                    
            }
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


                  

               }}}


                stage('Sonarqube Integeration') {
            when {
         expression { is_sonarqube == "Yes" }
     }
     steps {
         echo "Hello,sonarqube continue...!"
            script {
                def url = sh(returnStdout: true, script: 'git config remote.origin.url').trim()
                sh 'echo url'
                 withSonarQubeEnv(installationName: 'sonarqube-server', credentialsId: 'sonarqube-secret-token') {
                    

                     sh './gradlew sonarqube \
                     -Dsonar.projectName=${GIT_REPO_NAME} \
  -Dsonar.host.url=http://localhost:9000 \
      -Dsonar.projectKey=test  \
'
                     

                    
                }
         
                
                        
                        
                    
            }
          
        }
                }
       

              
          
                 stage("Quality Gate") {
            steps {
              timeout(time: 0, unit: 'SECONDS') {
                waitForQualityGate abortPipeline: true
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
        
    


