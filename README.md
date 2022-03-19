# Multi-Container Continuous Integration & Deployment into AWS 
In one of the previous posts I wrote about how we can achive Continuous Integration and Deployment into AWS. That post mainly talks about a single container which 
is hosted in NGINX container and deployed in AWS. 

You can see that here https://github.com/Kedarnadhsharma/DockerProductionWorkflowFrontend

However in a real world scenario, we would have multiple containers running and interacting with each other and also each of those services also need to deployed into 
Production independently. This post is a continuition of the previous one and involves deploying  multi containers to AWS.

## What is not changing

We would still use the same tools that we have leveraged in the previous post as below. 

1. React JS Library
2. Docker Desktop
3.Travis CI Account
4. GitHub
5. AWS Account

## What is changing

a. Previous post involves the code base to deployed into EBS (Elastic BeanStalk) and EBS creates the container images for us to run behind the NGINX.
In this post, rather than building the images, EBS pulls the images from Dockerhub and runs in Production.

b. I am going to add a new Express Server component (for API), Postgres DB and Redis Cache to simulate the below architecture.

![image](https://user-images.githubusercontent.com/50028950/159108063-66f34a5a-da76-4687-8612-24cb9bdd064c.png)
Reference : https://www.bogotobogo.com/DevOps/Docker/Docker-Compose-Nginx-Reverse-Proxy-Multiple-Containers.php

c. Both Postgres DB and Redis Cache will run as managed services within AWS and will be connected to Elastic Beanstalk

d. Connectivity needs to be established  b/w Elastic Beanstalk (where our application code is deployed), Postgres RDS, Redis Elastic Cache 

## Changes to Travis.yml file
As mentioned previously the current approach involves pushing the code to Dockerhub rather than EBS. 

![image](https://user-images.githubusercontent.com/50028950/159108313-b4638648-907c-4e23-a5d4-a2d958170749.png)

We need to add a after_success tag and this step involves two things Building the Docker image and then pushing the Docker image to my personal account in Docker hub
using Docker build and Docker Push commands.
Rest all looks exactly the same as before.

## Changes to ElasticBeanstack deployment

1. Elastic Beanstalk internally uses the docker compose file to deploy the app. Hence it needs to pull the images from the Docker hub. See the image names in the Docker hub

![image](https://user-images.githubusercontent.com/50028950/159108457-12a5055d-a3fa-462a-82d5-ff8d7368a2b8.png)

2. We need to pass the environment variables to the Docker Compose file during run time and EBS has to provide these details. 
3. In the AWS Elastic BeanStalk Console--> Configuration --> Software-->Edit. Supply the parameter values for each of the environment variables

![image](https://user-images.githubusercontent.com/50028950/159108565-8bd3acb3-580d-4df4-bab4-b65639103c52.png)

4. RedisHost and PGHost are the Endpoint URLs of the AWS Managed RDS and ElastiCache services that need to be created before. (Step c in "What is changing")
5. Establish Connectivity by creating a new Security Group in AWS and attaching the security group to all the 3 services (Elastic Beanstalk/RDS/ElasticCache).
I created a Security Group called dockercomplexsg and attached that to all 3 services.

RDS :
![image](https://user-images.githubusercontent.com/50028950/159108739-8e051708-681e-42fc-a9bf-ef984b8c238b.png)

ElastiCache :

![image](https://user-images.githubusercontent.com/50028950/159108788-7886b98e-8e1e-45ea-9122-48b77734f1a6.png)

ElasticBeanStalk:

![image](https://user-images.githubusercontent.com/50028950/159108870-6e0f65b2-59b7-4abc-ba34-e37442f119f2.png)

Now all these 3 services can communicate with each other using this Security group. 

Thats all!!. Make a change to the repo--> Triggers a new build in Travis CI --> Updates the DockerHub --> Deploys the code to AWS Beanstalk

Travis CI Build --> Pushes the Images to Docker Hub.

![image](https://user-images.githubusercontent.com/50028950/159108912-42fe2199-a35c-4936-8344-05fff157ffb8.png)

DockerHub : We can check the latest image details in the Docker hub

![image](https://user-images.githubusercontent.com/50028950/159108942-30a6464f-6c31-4188-aeee-30e20878fe19.png)

ElasticBeanstalk:

![image](https://user-images.githubusercontent.com/50028950/159108965-09b8ef97-46a6-4e50-be70-66a479f3fd68.png)

Finally when you browse the URL , you should see the React App and fetches some values from Redis Cache. 
If we click Submit button, it will call the express API to perform
some business logic as well.

![image](https://user-images.githubusercontent.com/50028950/159109019-94f08af9-1bf8-4bbe-9bc1-cad57e335929.png)

![image](https://user-images.githubusercontent.com/50028950/159109054-cb506d00-d9b3-4a0f-930e-020d7e671cb6.png)













