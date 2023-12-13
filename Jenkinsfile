node {
  def image
   stage ('checkout') {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/shushant-patra/java-app.git']]])      
        }
   
   stage ('Build') {
         def mvnHome = tool name: 'maven', type: 'maven'
         def mvnCMD = "${mvnHome}/bin/mvn "
         sh "${mvnCMD} clean package"           
        }
    stage('Build image') {
      sh "whoami"    
       app = docker.build("cicdproject-408006/springboot")
    }

    stage('Push image to gcr') {
        docker.withRegistry('https://gcr.io', 'gcr:gcp') {
            app.push("${env.BUILD_NUMBER}")
        }
    }

    stage('Update GIT') {
            script {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') 
                {
                    withCredentials([usernamePassword(credentialsId: 'shushant-patra', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        def encodedPassword = URLEncoder.encode("$GIT_PASSWORD",'UTF-8')
                        sh "git config user.email shushant.patra88@gmail.com"
                        sh "git config user.name shushant-patra"
                        sh "sed -i 's+cicdproject-408006/springboot.*+cicdproject-408006/springboot:${env.BUILD_NUMBER}+g' spring-boot.yaml"
                        sh "git add ."
                        sh "git commit -m 'jenkinsbuild: ${env.BUILD_NUMBER}'"
                        sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/javaapp.git HEAD:master"
                }
                    
                  }
                
            }
    }        
}
