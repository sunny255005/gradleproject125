import groovy.json.JsonSlurper

pipeline{
    environment {
        PROD_BRANCH = 'master'
        STAGING_BRANCH = 'staging'
        user_env_input = 'Development'
        is_unit_test_continue='No'
        is_sonarqube='No'
        GIT_REPO_NAME = GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
        
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
       

      stage("Quality Gate"){
            steps{
                     script{
        
                         def reportFilePath = "build/sonar/report-task.txt"
def reportTaskFileExists = fileExists "${reportFilePath}"
if (reportTaskFileExists) {
    echo "Found report task file"
    def taskProps = readProperties file: "${reportFilePath}"
    echo "taskId[${taskProps['ceTaskId']}]"
    while (true) {
        sleep 20
        def taskStatusResult    =
            sh(returnStdout: true,
               script: "curl -s -X GET -u admin:admin \'http:localhost:9000/api/ce/task?id=${taskProps['ceTaskId']}\'")
            echo "taskStatusResult[${taskStatusResult}]"
        def taskStatus  = new JsonSlurper().parseText(taskStatusResult).task.status
        echo "taskStatus[${taskStatus}]"
        // Status can be SUCCESS, ERROR, PENDING, or IN_PROGRESS. The last two indicate it's
        // not done yet.
        if (taskStatus != "IN_PROGRESS" && taskStatus != "PENDING") {
            break;
        }
    }
}
                     }}}
        
          
//             stage("Quality Gate"){
//                 steps{
//                     script{

//  def getURL = readProperties file: 'build/sonar/report-task.txt'
//  //sh 'echo ${getURL}'
//  sonarqubeURL = "${getURL['ceTaskUrl']}"
// // response=sh 'curl -k -s -X GET --url ${sonarqubeURL}'
// // echo "${response}"

//  echo "${sonarqubeURL }"
// //   timeout(time: 1, unit: 'MINUTES') { // Just in case something goes wrong, pipeline will be killed after a timeout
// //     def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
// //     if (qg.status != 'OK') {
// //       error "Pipeline aborted due to quality gate failure: ${qg.status}"
// //     }
// //   }
//   }
// }
            
stage("finished")
{
    steps{
        sh'echo finsihed'
    }
}

        }
}

        
    


