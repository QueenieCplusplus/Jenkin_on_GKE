# Jenkin_on_GKE

You now have access to Jenkins Serviceand a Kubernetes cluster managed by GKE. 
To take this solution further, you could use these components in your CD pipeline.

# Core Steps:

(1) create GKE cluster

(2) install Helm CLI

(3) install Jenkins

(4) connect to Jenkins Server

---------

# GKE

from step 1:

> Activate Cloud Shell

* 1.1, setup GCE zone.

      gcloud config set compute/zone <zone>
      
* 1.2, create GKE cluster.

      gcloud container clusters create j-ci-cd \
       --machine-type <machine-type> \
       --cluster-version <version> \
       --scopes "https://www.googleapis.com/auth/source.read_write,cloud-platform"
       // scopes enable Jenkins to access container registry and cloud source repository.
       
 * 1.3, check cluster is running.
 
       gcloud container clusters list
       
       [output]
       Name      location   master_version    master_ip   machine_type    node_version   num_node status
       j-ci-cd     <zone>    <version>      35.231.8.57   <machine type>    <version>    2        running
      
 * 1.4, cfrm that connectivity to cluster
 
        kubectl cluster-info
        
        [output]
        kubernets master is running at <ip> 
        backend is running at <ip/api/v1/proxy/namesapces/kube-system/services/default-http-backend>
        kubeDNS is running at <ip/api/v1/proxy/namesapces/kube-system/services/kube-dns>
   
  # Helm
   
  from step 2:
  
  > install and utile Helm CLI tool
  
  * 2.1, download Helm binary.
  
        wget https://get.helm.sh/helm-v3.2.1-linux-amd64.tar.gz
        
  * 2.2, unzip the Helm file.
  
        tar zxfv helm-v3.2.1-linux-amd64.tar.gz
        
  * 2.3, copy the file to local system.
  
        cp linux-amd64/helm
        
  * 2.4, add role to RBAC for adding jenkins permissions afterward.
  
        kubectl create cluster clusterrolebinding cluster-admin-binding 
              --clusterrole=cluster-admin\
              --user=$(gcloud config get-value account)
              
  * 2.5, add repo
  
         ./helm repo add jenkins-ci https://charts.jenkins.io
         ./helm repo update 
         ./helm version
         
 # Jenkins
 
 from step3:
 
> install and util Jenkins

* 3.1, use helm cli to install jenkins chart with yaml file.
 
      ./helm install cd-jenkins -f jenkins/values.yaml jenkinsci/jenkins 
          --version 2.6.4
          --wait
          
* 3.2, check the Pod installing Jenkins is at running state

        kubectl get pods
        
        [output]
        Name                  ready   status     AGE
        cd-jenkins-<code>      1/1    running     1m
        
* 3.3, forward port from cloud shell to jenkins UI.

      export POD_NAME=$(kubectl get pods 
      --namespace default 
      -l "app.kubernetes.io/component=jenkins-master" 
      -l "app.kubernetes.io/instance=cd-jenkins" 
      -o jsonpath="{.items[0].metadata.name}")
      kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &

* 3.4, check Jenkins Service is created well.

      kubectl get svc
      
      [output]
      Name               cluster-ip     external-ip    port(s)    gae
      cd-jenkins         <internal-ip1>   <none>      8080/tcp     3h
      cd-jenkins-agent   <internal-ip2>   <none>     50000/tcp     3h
      kubernets          <internal-ip3>   <none>       443/tcp     9h

* tips & attentions

   The Jenkins installation is using the Kubernetes Plugin to create builder agents. They will be automatically launched as necessary when the Jenkins master needs to run a build. When their work is done, they are automatically terminated and their resources are added back to the cluster's resource pool.
   
* 3.5, to retrieve the admins password and connect to Jenkins Server.

      printf $(kubectl get secret cd-jenkins 
              -o jsonpath="{.data.jenkins-admin-password}" 
              | base64 --decode)
              ;echo
              
* 3.6, to open Jenkins UI (chart), plz click on "Web Preview" tab.

* 3.7, log in by clicking on the "log in" tab, and enter the "admin" for user field, 
       and password for the password filed, then click on "log in" button.
       
        

