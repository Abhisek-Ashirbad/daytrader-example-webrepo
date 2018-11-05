

# DayTrader Web

	The Web UI to the DayTrader business functions



## Current State Architecture

DayTrader is multi-tier application built around the paradigm of an online stock trading system. Example business functions include
login, register, view portfolio, lookup stock quotes, and buy or sell stock. DayTrader was originally developed by IBM and donated 
to the Apache Geronimo community in the 2005 timeframe. The DayTrader architecture is representitive of monolithic applications that 
many businesses are still using. For that reason, we selected it to clarify the techniques for refactoring a monolith to microservices. 
For the original monolithic architecture, see http://geronimo.apache.org/GMOxDOC30/daytrader-a-more-complex-application.html. 



## Target State Architecture

![Target-State-Architecture](images/Target-State-Architecture.JPG)



### Web Application

The Web Application (highlighted in the green box) is a Spring Boot Application responsible for the UI to the DayTrader business functions. Although 
it still contains the business functions, their implementations have been changed to delegate their invocation to the appropriate microservice (API).




#### Static Viewpoint

![Static-Viewpoint](images/Static-Viewpoint.JPG)



#### Dynamic Viewpoint

![Dynamic-Viewpoint](images/Dynamic-Viewpoint.JPG)



#### Participant and Responsibilities

![Participant-Responsibilities](images/Participant-Responsibilities.JPG)



## Prerequisites

### Installation

#### For building Docker images and pushing them to a Docker registry

		1.	Inatall Java Development Kit (64 bit) 1.8+
		2.	Install Apache Maven 3.3.9+
		3.  Create DockerHub Account (https://hub.docker.com/)
		4.  Install Docker (https://www.docker.com/get-docker)
		
#### For deploying and managing applications on Kubernetes

		1.  Install kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl/)

#### For running Kubernetes on Minikube 

		1.  Install Minikube (https://kubernetes.io/docs/setup/minikube/)

#### For running Kubernetes on Amazon EKS

		1.  Create AWS Account (https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)
		2.  Install AWS IAM Authenticator (https://docs.aws.amazon.com/eks/latest/userguide/configure-kubectl.html)
		2.  Install AWS Command Line Interface (https://docs.aws.amazon.com/cli/latest/userguide/installing.html)
		3.  Install eksctl (https://github.com/weaveworks/eksctl)


### Configuration

#### Maven

        The following settings are required for Maven to push Docker images to you DockerHub account

        1.  Change the following property in daytrader-web/pom.xml to your DockerHub user name:
    
            <docker.image.prefix>${user.name}</docker.image.prefix>
               
        3.  Change the following key-value in daytrader-web/env/external/k8s/web-deployment.yaml to your DockerHub user name
        
            image: YOUR_DOCKERHUB_USERNAME/daytrader-gateway:4.0.18
        
        4.  The following property in daytrader-web/pom.xml is also worth noting. You do not have to change it, but if you do, change the above key-value accordingly.
        
    		<tag>4.0.18</tag>
        
        5.  Add the following settings to your Maven ~/.m2/settings.xml so Maven can push images to your DockerHub account

                <servers>
                  <server>
                        <id>docker.io</id>
                        <username>YOUR_DOCKERHUB_USERNAME</username>
                        <password>YOUR_DOCKERHUB_PASSWORD</password>
                    </server>
                </servers>       
   
        

#### AWS CLI

        1.  Configuring the AWS CLI (https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)

                
                
#### Kubernetes Cluster

        1.  You must have at least one Kubernetes cluster.

                a.  For Minikube,
        
                    $ minikube --memory 8192 --cpus 2 start
            
                    ## Minikube comes with a cluster. It was running out of memory so increased to 8GB of RAM. You may be able to run with less RAM, but that setting worked well.
            
                b.  For EKS, 
        
                    $ eksctl create cluster --name=user116-eks-cluster --region=us-east-1
            
                    ## This command creates an EKS cluster named user116-eks-cluster in the us-east-1 region. For additional details on parameters, see https://github.com/weaveworks/eksctl
            
        2.  If you installed Minikube and Amazon EKS, then you will have two contexts in your ~/.kube/config file. One of those contexts is for accessing the Minikube
            cluster; the other context is for accessing the EKS cluster. You must set the context to the cluster before running kubectl commands. Set the context once
            then all subsequent kubectl commands will be directed to that cluster. Set it again and the kubectl commands will go to the other cluster. To do that,
                  
                a.  For Minikube, 
                
                    $ kubectl config use-context minikube
                        
                b.  For Amazon EKS, 
                
                    $ kubectl config use-context user116-eks-cluster  ## Use the name of your Amazon EKS cluster

        3.  Now you can verify your cluster by running some kubectl commands
        
                a.  For Minikube
        
                    $ kubectl config use-context minikube
                    
                    Switched to context "minikube".

                    $ kubectl cluster-info
                    
                    Kubernetes master is running at https://192.168.99.100:8443
                    KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
                        
                b.  For Amazon EKS,
        
                    $ kubectl config use-context user116-eks-cluster  ## Use the name of your Amazon EKS cluster
                    
                    Switched to context "user116-eks-cluster".
            
                    $ kubectl kubectl cluster-info
                    
                    Kubernetes master is running at https://8D36DB7CBD2E3394FF0843CEA0C0A266.sk1.us-east-1.eks.amazonaws.com
                

   
## Delivery Pipeline


### Build Automation

		1.	cd daytrader-webapp
		2.	mvn clean install


### Functional Test (Spring Boot)

	The functional test is done in three phases: pre-functional-test functional-test, and post-functional-test

#### Phase 1: pre-functional-test

		1.	Start Accounts

			a.	cd daytrader-accountsapp\daytrader-accounts
			b.	env\external\bin\start_server.bat

		2.	Start Portfolios

			a.	cd daytrader-portfoliosapp\daytrader-portfolios
			b.	env\external\bin\start_server.bat

		3.	Start Quotes

			a.	cd daytrader-quotesapp\daytrader-quotes
			b.	env\external\bin\start_server.bat

		4. 	Start Gateway

			a.	daytrader-gatewayapp\daytrader-gateway
			b.	env\external\bin\start_server.bat

		5.	Start Web

			a.	cd daytrader-webapp\daytrader-web
			b.	env\external\bin\start_server

#### Phase 2: functional-test

		1.	Open: https://localhost:5443

			a.	Under The Configuration Tab

				- Press(Re)-create DayTrader Database Tables and Indexes
				- Press(Re)-populate DayTrader Database
				- Press Test Daytrader Scenario

			b.	Keep refreshing the browser until you are satisfied DayTrader is working correctly. 

				Note: each time you refresh the browser, it runs another functional test scenario. 

			That's it! You have now tested the daytrader

#### Phase 3: post-functional-test

		1.	Stop Accounts

			a.	cd daytrader-accountsapp\daytrader-accounts
			b.	env\external\bin\stop_server.bat

		2.	Stop Portfolios

			a.	cd daytrader-portfoliosapp\daytrader-portfolios
			b.	env\external\bin\stop_server.bat

		3.	Stop Quotes

			a.	cd daytrader-quotesapp\daytrader-quotes
			b.	env\external\bin\stop_server.bat

		4.	Stop Gateway

			a.	daytrader-gatewayapp\daytrader-gateway
			b.	env\external\bin\stop_server.bat

		5.	Stop Web

			a.	cd daytrader-webapp\daytrader-web
			b.	env\external\bin\stop_server



### Continuous Delivery

    Many of these manual steps can be automated by maven at a later date.
    

#### Create the Docker image and push it to DockerHub

        1.	$ cd daytrader-webapp
	    2.	$ mvn -Pcd clean install
		
	
#### Configure kubectl to access the right cluster
    
        1.  For Minikube,  
        
            $ minikube --memory 8192 --cpus 2 start
        
            $ kubectl config use-context minikube   
        
        2.  For EKS, 
        
            $ kubectl config use-context eks
            
            
#### Create the deployment, replica set, and pod(s) to run the application

	    1.  $ cd daytrader-webapp\daytrader-web\env\external\k8s
    
		2.	$ kubectl apply -f web-deployment.yaml 
		
		3.  $ kubectl get pods -o wide
		
		    ## Take note of the POD_NAME, and wait until the STATUS is Running.
              
        4.  $ kubectl logs {POD_NAME}
        
            ##  You should see : Tomcat started on port(s): 1443 (https)
        
        
        
#### Create the service so clients can communicate with pods via a constant IP

        Before creating the service it is important to understand that rationale for creating them. Remember that pods are ephemeral. If 
        a pod dies, then Kubernetes may create an identical pod but the pod will have a different IP. This means that clients should not 
        communicate with pods via the pod IP. Instead you should frontend all pods by a service, and clients should communicate with pods 
        through the service IP. 
        
        To create the service,

            1.  $ apply -f web-service.yaml
        
            2.  $ kubectl get services
        
                NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
                daytrader-web     ClusterIP   10.100.17.24     <none>        443/TCP         3d
       
                ## Notes
        
                a.  The above command creates a service proxy. The service proxy listens on the CLUSTER IP and PORT. It is backed by endpoints 
                    to one or more pods. When the service proxy receives an HTTPS request, it selects one of the endpoints (pods) and forwards 
                    it to the pod. The application running in the pod handles the request and returns an HTTPS response
            
                b.  There is no EXTERNAL-IP to the service. This means that you can't access the service from outside the cluster. That said,
                    you can assess it driectly from an application (or from a shell) that is running inside the pod, and indirectly by way of
                    the API Server. Both options are discussed below.
        
        Access the service directly,
            
            1.  $ kubectl get pods
        
                # Note the NAME of the web pod; you'll need it to execute the next command
        
            2.  $ kubectl exec <POD_NAME> -it - curl -k https://<CLUSTER-IP>/health
            
                {"status":"UP"}
  
        Access the service indirectly (via  the API Server)
                        
            1.  $ kubectl proxy
        
                Starting to serve on 127.0.0.1:8001
                
                ## Locates and authenticates to the API Server.

            2.  $ curl -k http://localhost:8001/api/v1/namespaces/default/services/https:daytrader-web:/proxy/health
        
                ## Accesses the web service through the API Server.       
        
        See also https://kubernetes.io/docs/concepts/services-networking/service/
            

        
#### Troubleshooting
        
    Review the standard output from your application using your pod names
    
        $ kubectl logs {POD_NAME} -f
        
        ## The -f option tails the logs
                
    Open a bash terminal into your pods

        $ kubectl exec -ti {POD_NAME} bash
        # echo $SERVER_PORT    ## you should see 1443
        # exit
        
    Use the kubernetes dashboard to manage and troubleshoot your cluster
    
        1.  Minikube
        
                a.  $ minikube dashboard
        
        2.  EKS (https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html)
        
                a.  $ kubectl proxy
                
                b.  $ kubectl -n kube-system get secrets | grep admin
                
                    ## Copy the NAME of the admin-user-token
                
                c.  $ kubectl -n kube-system describe secrets <ADMIN_USER_TOKEN>
                
                    ## Copy the value of the authentication token
                                    
                d.  Open the following link in a browser to access the dashboard
                
                    http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
                    
                e.  You will see a login screen; choose Token and then enter the <AUTHENTICATION_TOKEN> from step c. 
   
        See also https://github.com/AcalephStorage/kubernetes-dashboard/blob/master/docs/user-guide/troubleshooting.md
                
                



### Functional Test (Kubernetes)

    The functional test is done in three phases: pre-functional-test functional-test, and post-functional-test


#### Phase #1: Pre-Functional Test
    
        Deploy microservies per their Continuous Delivery instructions. See the README.MD files in the following repos
        
            1.  daytrader-example-accountserepo
            2.  daytrader-example-gatewayrepo
            3.  daytrader-example-portfoliosrepo
            4.  daytrader-example-quotesrepo
            5.  daytrader-example-webrepo
               
                
#### Phase #2: Functional Test

        For Minikube,  
        
            1.  $ minikube --memory 8192 --cpus 2 start
            
            2.  $ kubectl config use-context minikube
            
            2.  $ kubectl cluster-info
                                    
                Kubernetes master is running at https://192.168.99.100:8443
            
                ## Note the IP address of the Kubernetes master (192.168.99.100).
                
            4.  $ kubectl get services daytrader-web
            
                NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
                daytrader-web          NodePort    10.98.28.179     <none>        443:31198/TCP   9d

                ## The PORT(S) column shows the internal port of the cluter IP (443) and the node port (31198)
        
    		5.	Open: https://192.168.99.100:{NODE_PORT}
    			
            	a.	Under The Configuration Tab

        			- Press(Re)-create DayTrader Database Tables and Indexes
		        	- Press(Re)-populate DayTrader Database
	        		- Press Test Daytrader Scenario

			    b.	Keep refreshing the browser until you are satisfied DayTrader is working correctly. 

				Note: each time you refresh the browser, it runs another functional test scenario. 

			That's it! You have now tested the daytrader
        
        For EKS, 
        
            1.  $ kubectl config use-context eks
            
            2.  $ kubectl cluster-info
                                    
                Kubernetes master is running at https://8D36DB7CBD2E3394FF0843CEA0C0A266.sk1.us-east-1.eks.amazonaws.com
            
                ## Note the IP address of the Kubernetes master.
                
            3.  $ kubectl proxy
            
                Starting to serve on 127.0.0.1:8001 
            
                ## This command starts a reverse proxy for locating the apiserver and authenticating for and on behalf of the browser
                            
    		5.	Open your browser and navigate to: http://localhost:8001/api/v1/namespaces/default/services/https:daytrader-web:/proxy/
    			
            	a.	Under The Configuration Tab

        			- Press(Re)-create DayTrader Database Tables and Indexes
		        	- Press(Re)-populate DayTrader Database
	        		- Press Test Daytrader Scenario

			    b.	Keep refreshing the browser until you are satisfied DayTrader is working correctly. 

				Note: each time you refresh the browser, it runs another functional test scenario. 

			That's it! You have now tested the daytrader
        
        
#### Phase #3: Post-Functional Test

        (Optional)
                          
        $ kubectl delete all --all    
    
    


## API Endpoints 

### Swagger UI Documentation

	To see the API documentation, 

		1. Start the application

			a.	cd daytrader-webapp
			b. 	daytrader-web\env\external\bin\start_server

		2. 	Point you browser to https://localhost:2443/swagger-ui.html

		3.	Stop the application after reviewing the API documentation

			a.	cd daytrader-webapp	
			b.	daytrader-web\env\external\bin\stop_server

	