podTemplate(label: 'jnlp-slave', cloud: 'kubernetes', containers: [
    containerTemplate(
        name: 'jnlp', 
        image: '192.168.31.61/library/jenkins-slave', 
        alwaysPullImage: true 
    ),
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    hostPathVolume(mountPath: '/usr/bin/docker', hostPath: '/usr/bin/docker'),
  ],
  imagePullSecrets: ['registry-pull-secret'],
) 
{
  node("jnlp-slave"){
      stage('Git Checkout'){
         checkout([$class: 'GitSCM', branches: [[name: '${Tag}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '2da2a0db-b3bc-4158-b2aa-407a7f1e0842', url: 'git@192.168.31.61:/home/git/tomcat-java-demo']]])
      }
      stage('Unit Testing'){
      	echo "Unit Testing..."
      }
      stage('Maven Build'){
          sh "mvn clean package -Dmaven.test.skip=true"
      }
      stage('Build and Push Image'){
          sh '''
          Registry=192.168.31.61
          docker login -u admin -p Harbor12345 $Registry 
          docker build -t $Registry/project/demo:${Tag} -f Dockerfile .
          docker push $Registry/project/demo:${Tag}
          '''
      }
      stage('Deploy to K8S'){
          sh '''
          sed -i "/demo/{s/latest/${Tag}/}" deploy.yaml
          sed -i "/namespace/{s/default/${Namespace}/}" deploy.yaml
          ''' 
          kubernetesDeploy configs: 'deploy.yaml', kubeConfig: [path: ''], kubeconfigId: '74bbb361-e426-4b66-ae41-c4cbceadd809', secretName: '', ssh: [sshCredentialsId: '*', sshServer: ''], textCredentials: [certificateAuthorityData: '', clientCertificateData: '', clientKeyData: '', serverUrl: 'https://']
      }
      stage('Testing'){
          echo "Testing..."
      }
  }
}
