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
name=$(echo $template | cut -d/ -f2)
catalog="EEA:$name"
rancher --url $RANCHER_URL --access-key $RANCHER_ACCESS --secret-key $RANCHER_SECRET --env $RANCHER_ENVID catalog refresh | grep $catalog 
check=$(rancher --url $RANCHER_URL --access-key $RANCHER_ACCESS --secret-key $RANCHER_SECRET --env $RANCHER_ENVID stack | grep $stack_id)
current_catalog=$(echo $check | awk '{print $4}')
upgrade=$(echo $check | awk '{print $7}')
if [ "$current_catalog" == "catalog://EEA:$name:$number" ]; then echo "Stack already upgaded to the latest release"; exit 0; fi
count=0
while [ "$upgrade" != "$catalog:$number" ] && [ $count -lt 30 ]; do
    echo "did not find stack to be upgrade-able yet, sleeping 1 min"
    sleep 60
    rancher --url $RANCHER_URL --access-key $RANCHER_ACCESS --secret-key $RANCHER_SECRET --env $RANCHER_ENVID catalog refresh | grep $catalog 
    let count=$count+1
    upgrade=$(rancher --url $RANCHER_URL --access-key $RANCHER_ACCESS --secret-key $RANCHER_SECRET --env $RANCHER_ENVID stack | grep $stack_id | awk '{print $7}')
done
if [ $count -eq 30 ]; then echo "In 30 minutes stack is not upgrade-able, exiting"; exit -1; fi
echo "Upgrading stack"
rancher --url $RANCHER_URL --access-key $RANCHER_ACCESS --secret-key $RANCHER_SECRET --env $RANCHER_ENVID catalog upgrade catalog://$catalog:$number --stack $stack_id '''

         }
        }
      }
    }
	  
  }
	
}
