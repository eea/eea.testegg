#!groovy

pipeline {
  agent any

  environment {
        GIT_NAME = "eea.testegg"
        template = "templates/rabbitmq"
	stack_id = '1st1824'
    }

  stages {
   
    stage('Test') {

      steps {
        node(label: 'docker') {
          withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'Rancher_dev_token', usernameVariable: 'RANCHER_ACCESS', passwordVariable: 'RANCHER_SECRET'],string(credentialsId: 'Rancher_dev_url', variable: 'RANCHER_URL'),string(credentialsId: 'Rancher_dev_envid', variable: 'RANCHER_ENVID')]) {
            sh '''rm -rf eea.rancher.catalog
	    git clone https://github.com/eea/eea.rancher.catalog.git
cd eea.rancher.catalog/$template
export number=$(find . -maxdepth 1 -type d | awk  'BEGIN{FS="/"}{print $2}' | sort -n | tail -n 1)
cd ..
rm -rf eea.rancher.catalog
echo  "Number is $number"
rancher --url $RANCHER_URL --access-key $RANCHER_ACCESS --secret-key $RANCHER_SECRET --env $RANCHER_ENVID catalog refresh '''
          }
        }
      }
    }
	  
  }
	
}
