podTemplate(label: 'esjenkinspod',
  containers: [ 
    containerTemplate(
      name: 'docker', 
      image: 'docker', 
      command: 'cat', 
      resourceRequestCpu: '100m',
      resourceLimitCpu: '300m',
      resourceRequestMemory: '300Mi',
      resourceLimitMemory: '500Mi',
      ttyEnabled: true
    ),
    containerTemplate(
      name: 'kubectl', 
      image: 'amaceog/kubectl',
      resourceRequestCpu: '100m',
      resourceLimitCpu: '300m',
      resourceRequestMemory: '300Mi',
      resourceLimitMemory: '500Mi', 
      ttyEnabled: true, 
      command: 'cat'
    ),
    containerTemplate(
      name: 'helm', 
      image: 'alpine/helm:latest', 
      resourceRequestCpu: '100m',
      resourceLimitCpu: '300m',
      resourceRequestMemory: '300Mi',
      resourceLimitMemory: '500Mi',
      ttyEnabled: true, 
      command: 'cat'
    )
  ],

  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    hostPathVolume(mountPath: '/usr/local/bin/helm', hostPath: '/usr/local/bin/helm')
  ]
  ) {
    node('esjenkinspod') {
      stage('Checkout') {
        echo 'Checkout repository from GitHub'
        checkout scm
      }
      stage('Build') {
        echo 'Build elasticsearch application'
        container('helm') { 
          sh 'helm repo add elastic https://helm.elastic.co'
          sh 'helm repo update'
        }
        
      }
      stage('Test') {
          container('docker') {  
            sh 'hostname'
            sh 'hostname -i' 
            sh 'docker ps'
            sh 'ls'
          }
          container('kubectl') {
            echo 'Test little ELK stack'
            sh 'kubectl get pods -n default'  
          }
          container('helm') { 
            sh 'helm repo update'

            // Deploy to namespace according to branch
            if (env.BRANCH_NAME.equals("dev")) {
                env.NAMESPACE="dev-test"
                
            } else {
                env.NAMESPACE="prod-test"
            }
            echo "Testing in namespace: ${env.NAMESPACE}"

            // sh 'helm upgrade --install elasticsearch elastic/elasticsearch -f elasticsearch/values.yaml -n ${env.NAMESPACE} --create-namespace'
            sh 'helm upgrade --install logstash elastic/logstash -f logstash/values.yaml -n ${env.NAMESPACE} --create-namespace'
            sh 'helm upgrade --install kibana elastic/kibana -f kibana/values.yaml -n ${env.NAMESPACE} --create-namespace'
            // sh 'helm upgrade --install metricbeat elastic/metricbeat -f metricbeat/values.yaml -n ${env.NAMESPACE} --create-namespace'

            // Test deployed pods
            // sh 'helm test elasticsearch'
            sh 'helm test logstash'
            sh 'helm test kibana'
            // sh 'helm test metricbeat'

            // sh 'helm uninstall elasticsearch -n $NAMESPACE'
            sh 'helm uninstall logstash -n ${env.NAMESPACE}'
            sh 'helm uninstall kibana -n ${env.NAMESPACE}'
            // sh 'helm uninstall metricbeat -n $NAMESPACE'
          }
      }
      stage('Deploy') {
        echo 'Deploy elasticsearch application'
        container('helm') { 
            sh 'ls'

            if (env.BRANCH_NAME.equals("dev")) {
                env.NAMESPACE="dev"
                
            } else {
                env.NAMESPACE="prod"
            }
            echo "${env.NAMESPACE}"

            // sh 'helm upgrade --install elasticsearch elastic/elasticsearch -f elasticsearch/values.yaml -n prod --create-namespace'
            // sh 'helm upgrade --install logstash elastic/logstash -f logstash/values.yaml -n prod --create-namespace'
            // sh 'helm upgrade --install kibana elastic/kibana -f kibana/values.yaml -n prod --create-namespace'
            // sh 'helm upgrade --install metricbeat elastic/metricbeat -f metricbeat/values.yaml -n prod --create-namespace'
        }
      }      
    }
}
