podTemplate(
    inheritFrom: 'jnlp',
    containers: [
        containerTemplate(name: 'docker', image: 'docker:18.02', ttyEnabled: true, command: 'cat')
    ],
    volumes: [
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
    ]
)

{
node(POD_LABEL) {
  def GIT_HASH = checkout(scm).GIT_COMMIT
  def credArr = []
  def buildEnvs = []

  echo "Set vars for dev."
     credArr = [
       string(credentialsId: 'ARGOCD_DEV-US-EAST2-OAK9-CLOUD', variable:'ARGOCD_AUTH_TOKEN')
     ]

     buildEnvs = [
       "ECR_IMAGE_NAME=demo-image",
       "ARGOCD_SERVER=argogrpc.argocd.svc.cluster.local",
       "COMMIT_HASH=${GIT_HASH}",
       "ENVIRONMENT=dev"
       ]
   withCredentials(credArr) {
    withEnv(buildEnvs) {
      stage('Build Docker Image') {
        container('docker') {
          echo "Building docker image...${ECR_IMAGE_NAME}:${COMMIT_HASH}"
          statusCode = sh(
             script: '''#!/bin/bash -e
                       IMAGE_NAME_TAG=${ECR_IMAGE_NAME}:${COMMIT_HASH}
                       IMAGE_NAME=${ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/$IMAGE_NAME_TAG
                       echo $IMAGE_NAME_TAG
                       aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
                       docker build -t $IMAGE_NAME_TAG --network=host -f ./Dockerfile . | while read line ; do echo "\$(date)| \$line"; done;
                       docker tag $IMAGE_NAME_TAG \$IMAGE_NAME
                       docker push \$IMAGE_NAME
                    ''',
            returnStatus: true
          )
          sh "echo ${statusCode}"
          if(statusCode >= 1) {
            error('Docker Build error')
          }
        }
      }
      stage('Run ARGOCD') {
        container('docker') {
          echo "Forcing refresh of repo."
          statusCode = sh(
             script: '''#!/bin/bash -e
                       IMAGE_NAME_TAG=${ECR_IMAGE_NAME}:${COMMIT_HASH}
                       SSM_NAME=/jenkins/${ECR_IMAGE_NAME}_${ENVIRONMENT}
                       IMAGE_NAME=${ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/$IMAGE_NAME_TAG
                       aws ssm put-parameter --name $SSM_NAME --value $COMMIT_HASH --type String --overwrite
                       
                       curl --insecure https://${ARGOCD_SERVER}/download/argocd-linux-amd64 > ./argocd
                       chmod +x ./argocd
                       ./argocd --insecure --grpc-web app get demo-dev-us-east2-oak9-cloud --hard-refresh
                       ./argocd --insecure --grpc-web app sync demo-dev-us-east2-oak9-cloud --force
                       ./argocd --insecure --grpc-web app wait demo-dev-us-east2-oak9-cloud --timeout 600
                       ''',
            returnStatus: true
          )
          if (statusCode && statusCode != 2 && statusCode != 255) {
             error('error')
          }                
        }
      }
    }
   }
  }
}