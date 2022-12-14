pipeline 
{
  agent
  {
      label 'maven'
  }
         stages
        {
            stage('BUILD')
            {
                steps
                {
                    echo 'building'
                    sh 'mvn clean package'
                }
            }
            
            
            
                  stage('Create Container Image') 
                  {
                    steps 
                    {
                         echo 'Create Container Image..'
        
                        script 
                        {

                          openshift.withCluster() { 
                               openshift.withProject("helloworld") {
  
                             def buildConfigExists = openshift.selector("bc", "spring-boot").exists() 
    
                          if(!buildConfigExists){ 
                          openshift.newBuild("--name=spring-boot", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
                             } 
    
                       openshift.selector("bc", "spring-boot").startBuild("--from-file=target/jb-hello-world-maven-0.2.0-SNAPSHOT.jar", "--follow") 
                                   
                               } }

                             }
                      }
                  }
        
          stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
            openshift.withProject("helloworld") { 
              def deployment = openshift.selector("dc", "spring-boot") 
     
              if(!deployment.exists()){ 
                 openshift.newApp('spring-boot', "--as-deployment-config").narrow('svc').expose() 
                  } 
    
                      timeout(5) { 
                           openshift.selector("dc", "spring-boot").related('pods').untilEach(1) { 
                           return (it.object().status.phase == "Running") 
                                 } 
                            } 
                         } 
                      }
        }
         }
                }
}

}

