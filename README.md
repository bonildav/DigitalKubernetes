##############################################
Author: David Bonilla
Created: 20/05/2019
Description: Postgres deployment with email alerting
##############################################

Requirements:
1: Write a kubernetes object that will initialize a postgres instance with the below configuration
  user: postgres
  password: Aw3s0m3
  Database: WORKSHOP
  
2: Send email notification every time the pod is stopped 

3: Bonus: create a table already inside the DB

##############################################
General Anylysis:
1 - Deploy the postgres container and deploy any monitoring tool to catch when the pod will stop (Eg: Prometheus,Stackdriver,appdynamycs,relyc,etc...) This will be the most common enterprise solution from my perspective.
2 - Deploy the postgres container with heirloom-mailx and use the smntp service from google to send emails and add a prestop handler in order to send an email before lost the pod. 
################################################

This document will follow the second option.

requirements:
kubernetes cluster, kubectl command-line tool, docker engine. 

                                                  -step 1 -
create a docker file to meet the requirements, you can review the details in the "Docker file" in sumary will contain the postgres deployment with the requested configurationnd and heirloom-mailx:
  - user: postgres
  - password: Aw3s0m3
  - Database: WORKSHOP
  - create a table already inside the DB

in order to create the image from the docker command run the below command:
docker build -t digital:v4 .
note: make sure you are in the same path of the Dockerfile

                                                  -step 2 -
Once we have the image I submit to any repository where you will pull the image in my case I pushed to "container registry" from google cloud" 

or

you can pull from if you want to avoid previous steps 
docker pull gcr.io/formal-guru-208519/digital:v4
docker pull gcr.io/formal-guru-208519/digital@sha256:12982026f4d70ebc20bcb3b449e317edbe474d388f0149019bbd74bad25eee7a

                                                  
                                                  -step 3 -
Modify email value in deployment-posgres.yamls in order to get the emails
exec:
    command: ["/bin/sh", "-c" , "echo POD-EMERGENCY | mailx -v -s POD-WARNING -S smtp-use-starttls -S ssl-verify=ignore -S smtp-auth=login -S smtp=smtp://smtp.gmail.com:587 -S from=david.digital.test2@gmail.com -S smtp-auth-user=david.digital.test2@gmail.com -S smtp-auth-password=19821982DBc -S ssl-verify=ignore -S nss-config-dir=/etc/pki/nssdb/  <email@digitalonus.com>"]

note:this is not the best approach for an enterprise solution since we have the password as part of the command but for practical execution decided to do it on that way.
                                                  
                                                  -step 5 - 
Run deployment-posgres.yamls in your kubernetes cluster 

kubectl create -f deployment-posgres.yamls

This will deploy 3 components:
-Persistant volume
-Deployment 
-LoadBalancer to expose our db

                                                  -step 6 -
Once we validate our pod and the deployment its healthy, lets test the "Aw3s0m3" password
Run
kubectl get service postgres-digital-lb

take the external ip generated by the service and run the below command:

psql -h <ExternalIP> -U postgres --password -p 5432 WORKSHOP
  
after be connected to the WORKSHOP DB lets validate a table already created with content 
run
\c postgres
type password if need it 
run
select * from games;
you will see an output like this:

postgres=# select * from games;
      game      | category
----------------+----------
 Chrono Trigger | (RPG)
(1 row)

Type: ctrl + d in order to exit.

                                                  
                                                  -step 7 -
Lets validate pods notification,run the below command
kubectl get pods --sort-by=postgres-digital

then run
kubectl delete pod <pod name>
  
you will get a notification to the changed email from david.digital.test2@gmail.com                       
                                                  
                                                  -step 8 -
Clean all the deployment                                        
                                                  
kubectl delete -f deployment-posgres.yamls


Many thanks many things I could do it better unfortunately my job and the deadline killed me. 

I´ll really appreciate your feedback.


 
                                                                               






