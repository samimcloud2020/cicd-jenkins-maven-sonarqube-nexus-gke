***********PROJECT ON CI/CD PIPELINE ON JENKINS + MAVEN + NEXUX + SONARQUBE + DEPLOY GKE CONTAINER***********
This project Source code: on Java help of Maven project

Jenkins to BUILD, TEST, VERIFY etc and make war file & trigger for
source code test in Sonarqube and if Quality gates passed then the war file pushed to 
Nexus repository and by help of executing instruction in war file to make a dockerfile 
and made image by docker buld and then from image create a container and deplo in GKE 
stagging cluster and wait for SRE approval manually and if stagging cluster verified ok 
then SRE engineer approved it and it deployed to Production cluster.

one Deploy.yaml file is deployed with Container image in kubernetes cluster and deploy.yaml 
having deployment of the image and a service is deployed and User accessed by LOADBALANCER url.

***********************************************************************************************************
