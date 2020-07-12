node {
// ws("workspace/${env.JOB_NAME}"){
      def serviceName
      def taskFamily
      def dockerFilePrefix
      def clusterName

      def remoteImageTag  = "13" //"${BUILD_NUMBER}"
      def dir = "$WORKSPACE"
      def taskDefile      = "file://${dir}/aws/task-definition-${remoteImageTag}.json"
      def ecRegistry      = "https://%ACCOUNT%.dkr.ecr.ap-southeast-2.amazonaws.com"

    stage('code checkout') {
        // checkout([$class: 'GitSCM', branches: [[name: 'develop']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: false, timeout: 20]], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'JK-Bitbucket', url: 'https://jkothuru@bitbucket.org/dcmdevs/api-vpn-setup.git']]])
           checkout scm
   }
    /*
    stage('docker build') {

        docker.build("042235167771.dkr.ecr.ap-southeast-2.amazonaws.com/api-vpn-setup:$BUILD_NUMBER")

        }
    
    stage('docker push') {

        sh ' eval \$(aws ecr get-login --no-include-email --region ap-southeast-2)'
        sh 'docker push 042235167771.dkr.ecr.ap-southeast-2.amazonaws.com/api-vpn-setup:$BUILD_NUMBER'
        echo "Pushed image to ecr repository"
    }
*/
      stage("Deploy") {
  
        sh  "                                                                     \
          sed -e  's;%BUILD_TAG%;${remoteImageTag};g'                             \
                  aws/fargate-task.json >                                      \
                  aws/task-definition-${remoteImageTag}.json                      \
        "

        // Get current [TaskDefinition#revision-number]
        def currTaskDef = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs describe-task-definition  --task-definition ${taskFamily}     \
                                              | egrep 'revision'                  \
                                              | tr ',' ' '                        \
                                              | awk '{print \$2}'                 \
          "
        ).trim()

        def currentTask = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs list-tasks  --cluster ${clusterName}                          \
                                --family ${taskFamily}                            \
                                --output text                                     \
                                | egrep 'TASKARNS'                                \
                                | awk '{print \$2}'                               \
          "
        ).trim()

        if(currTaskDef) {
          sh  "                                                                   \
            aws ecs update-service  --cluster ${clusterName}                      \
                                    --service ${serviceName}                      \
                                    --task-definition ${taskFamily}:${currTaskDef}\
                                    --desired-count 0                             \
          "
        }
        if (currentTask) {
          sh "aws ecs stop-task --cluster ${clusterName} --task ${currentTask}"
        }

        sh  "                                                                     \
          aws ecs register-task-definition  --family ${taskFamily}                \
                                            --cli-input-json ${taskDefile}        \
        "

        
        def taskRevision = sh (
          returnStdout: true,
          script:  "                                                              \
            aws ecs describe-task-definition  --task-definition ${taskFamily}     \
                                              | egrep 'revision'                  \
                                              | tr ',' ' '                        \
                                              | awk '{print \$2}'                 \
          "
        ).trim()
        
        if(!currTaskDef) {
          sh  "                                                                   \
            aws ecs update-service  --cluster ${clusterName}                      \
                                    --service ${serviceName}                      \
                                    --task-definition ${taskFamily}:${currTaskDef}\
                                    --desired-count 0                             \
                                    --launch-type "FARGATE"                       \
                                    --network-configuration 'awsvpcConfiguration={subnets=[subnet-0ad93e27d3491ab33],securityGroups=[sg-0d75fdcec73061038],assignPublicIp=ENABLED}' \
                                    --region ap-southeast-2"                       \
        }
        sh  "                                                                     \
          aws ecs update-service  --cluster ${clusterName}                        \
                                  --service ${serviceName}                        \
                                  --task-definition ${taskFamily}:${taskRevision} \
                                  --desired-count 1                               \
        "
      }

      stage("BUILD SUCCEED") {
        echo "Deployment Success"
      } 
}
// }
