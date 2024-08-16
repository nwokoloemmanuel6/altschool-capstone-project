![](./images/sock-shop%20on%20%20browser.jpg)

# DEPLOYMENT OF THE SOCK-SHOP APPLICATION AND PROMETHEUS ON EKS USING TERRAFORM AND JENKINS

Terraform is a great IaC tool that is going to be used to deploy an eks cluster with two worker nodes and auto scaling.

Terraform will also be used to deploy the sock-shop application, Prometheus and nginx ingress controller on the eks cluster.. 

I am going to explain what every directory is meant for so you do not get confused.


## WHAT MAKES IT WORK?

In the repository above there are two directories and three files and I am going to be explaining to you how they work together… sit tight




- ### THE EKS DIRECTORY

The **eks directory** is the first directory on the repository and it contains all our code in terraform to spin up our eks cluster.. but not just the eks cluster.. 

we are also setting up the environment where the eks cluster is going to run. For instance we are creating the vpc, subnets, internet gateway, eks roles, node group roles and also a bastion host if you choose to use the private instance..



- ### THE KUBERNETES DIRECTORY

Now this directory is split into fours sub directory:

1. **Ingress-rule**
2. **Micro-service**
3. **Nginx-controller**
4. **Prometheus-helm**


Let’s dive deep into each of this!

#### 1. **MICRO-SERVICE**

The **micro-service directory** holds all the code that would be used to deploy the sock-shop application. And there are 10 services that make up this micro-service 

No changes needs to be made to this file as the codes are exactly the way it is on their GitHub repository. It was only converted to terraform.


#### 2. **PROMETHEUS-HELM**


The **Prometheus-helm directory** holds all the code that would be needed to install Prometheus and grafana to our eks cluster for monitoring.

The values.yaml file is the default configuration of Prometheus. If you want Prometheus to behave in a certain way the that’s where you go.


#### 3. **NGINX-CONTROLLER**

The **nginx-controller directory** is where we provide the code in terraform to install our nginx-ingress controller that would help us route traffic within our cluster 

Also in this directory we are creating our route 53 hosted zone with CNAME records for grafana and sock-shop

You can make a change to the domain name used in the variables.tf file if you have a live domain you want to use.

Lastly we are also requesting an SSL certificate from ACM (Amazon certificate manager) for our domain names and sub domains respectively 


#### 4. **INGRESS-RULE**

The ingress-rule directory holds the code in terraform that would be used to create our ingress rule and route the traffic to our nginx ingress load balancer which will the route the traffic to the selected service.

In this case there are just two - the micro-service (sock-shop) and grafana - you can also observe that they are both opened on port 80 in both files

Here also is where you make a change to the domain name in the host name section to:

```bash script
sock-shop.yourdomain.com
```

In both files..




- ### **THE JENKINS FILES**

There are two Jenkins files present in this repository.

- cluster-Jenkinsfile
- Jenkinsfile

####  **CLUSTER-JENKINSFILE**

As you can tell that the cluster-Jenkinsfile file is created for Jenkins to help us deploy the eks cluster first 


#### **JENKINSFILE**

The Jenkinsfile is where the code to run everything in the kubernetes directory is and this is ran only after the cluster-Jenkinsfile is done running 

It creates all the resources in this manner:

```
Prometheus > microservice > ingress-rule > nginx-ingress controller 
```

And this is because it take a while for the acm in the kubernetes/nginx-ingress directory to completely run. That’s why we are running it last..


#### **INSTALLER.sh**

This is a script file that helps you setup the environment for your newly launched instance 

It install all the prerequisites like Jenkins, Kubectl, AWS cli and terraform



## HOW TO RUN IT

1. First off you need to create an ec2 t2.medium instance where we are going to run our installer.sh


2. Git clone this repository  and cd into it

```
git clone {repo url} && cd 
```


3. Make the installer.sh file executable and run the script.

```
chmod +x installer.sh
```

4. Now go to your browser and put in the instance ip address on port 8080

```
youripaddress:8080
```

5. Then create and configure your Jenkins user with an Access key and a Secret access key (use the variable that was provided in the Jenkinsfile to avoid error)

![access key](./images/jenkins%20access%20keys.jpg)


6. Now you can create your pipeline for the eks cluster

![eks pipeline](./images/eks%20ran.jpg)

7. Then do the same for the kubernetes directory 

![sock-shop pipeline](./images/sock-shop%20ran.jpg)

8. When done running go over to route 53 on AWS select your hosted zone and copy the 4 name servers that would be provided over to you domain provider and update it

![route 53](./images/nameserver%20on%20route53.jpg)

9. Now wait for about 2 hours then take your domain name with this sub domain and paste it your browser 

```
Sock-shop.yourdomain.con

grafana.yourdomain.com
```
![grafana](./images/grafana-dashboard%20on%20browser.jpg)

![sock-shop](./images/sock-shop%20on%20%20browser.jpg)

You should be able to see your grafana dashboard and sock-shop application running 


## DESTROYING IT

Destroying it is also quite easy.. on your Jenkins page click on the build with parameters and select destroy instead of create..

![sock-shop destroy pipelind](./images/destroy%20sock-shop.jpg)

Make sure you destroy the sock-shop first before destroying the eks cluster


Okayy that’s it. Congratulations you just learned how to deploy an application on eks using Jenkins! 

Star this repository if you loved every line you read!