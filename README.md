# Qwiklabs Set Up and Configure a Cloud Environment in Google Cloud Challenge Lab                                        
Step by Step guide to solve this challenge. Can also refer my video for the same : https://youtu.be/h-hz12o_qvs                  

# Task 1: Create development VPC manually                           
Navigation menu->VPC Network->Create VPC Network->                                   
Name : griffin-dev-vpc                                   

New Subnet                              
Name : griffin-dev-wp                                   
Region : us-east1                                  
IP address range : 192.168.16.0/20                             
Done                                
Add Subnet                                      
Name : griffin-dev-mgmt                                    
Region : us-east1                             
IP address range : 192.168.32.0/20                                       
Done                            

Create                                    

Check my Progress                                     

# Task 2: Create production VPC manually                            
Navigation menu->VPC Network->Create VPC Network->                                 
Name : griffin-prod-vpc                                

New Subnet                            
Name : griffin-prod-wp                        
Region : us-east1                             
IP address range : 192.168.48.0/20                                 
Done                              
Add Subnet                                      
Name : griffin-prod-mgmt                        
Region : us-east1                                     
IP address range : 192.168.64.0/20                                        
Done                                        
 
Create                                   

Check my Progress                                

# Task 3: Create bastion host
Navigation menu->Compute Engine->VM Instances->Create                                 
Name : griffin-dev-db                                              

Region : us-east1                                          
Zone : us-east1-b                             

Networking              
Network tags : bastion                                                                                         
Network interfaces : griffin-dev-mgmt                                              

Add network Interface                                          
Network : griffin-prod-vpc                  
Subnetwork :griffin-prod-mgmt                       

Leave rest all value to default                                   
Done                            

Create                                           
 
Navigation menu->VPC Network->Firewall->Create firewall rule                               

Name : allow-bastion-dev-ssh                                   
Network : griffin-dev-vpc             
Targets : Specified target tags                         
Target tags : bastion                       
Source filter : IP ranges                        
Source IP ranges : 192.168.32.0/20                                      
Specified Protocols and ports : check tcp: 22                                     
Create                                             
 
Create a Firewall Rule                                
Name : allow-bastion-prod-ssh                                
Network : griffin-prod-vpc                          
Targets : Specified target tags                         
Target tags : bastion                          
Source filter : IP ranges                                       
Source IP ranges : 192.168.48.0/20                                      
Specified Protocols and ports : check tcp: 22                                    
Create                                                 
 
Check my Progress                                    

Task 4: Create and configure Cloud SQL Instance                                   
Navigation Menu->SQL->Create Instance->Choose MySQL                           
Name:- griffin-dev-db                                       
Region:- us-east1                               
Zone:- us-east1-b                       
Root password:- e.g. 12345                              
Create                                   

Under Connect to instance                                    
Select Connect using Clod Shell                                

Cloud Shell                                   
Run the following command                                       
->gcloud sql connect griffin-dev-db --user=root --quiet                                             
                                                                 
Enter root password : created in previous step [e.g 12345]                                    

->CREATE DATABASE wordpress;                                       
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";                                   
FLUSH PRIVILEGES;                                                                           
                         
->exit                                         

Check my Progress                                   

# Task 5: Create Kubernetes cluster                               
Navigation menu->Kubernetes Engine->Clusters->Create Cluster                     
Name : griffin-dev                                           
Zone : us-east1-b                                 

In the left menu ->NODE POOLS                   
Click default pools                    
Size                            
Number of Nodes : 2                          

In the left menu ->NODE POOLS                         
Click Nodes                           
Series : N1                             
Machine type : n1-standard-4                             

In the left menu ->CLUSTER                                   
Click Networking                               
Network : griffin-dev-vpc                    
Node subnet : griffin-dev-wp                           

Create                                    
                 
Check my progress                                       

# Task 6: Prepare the Kubernetes cluster                               

Cloud Shell                                             
->gsutil cp -r gs://cloud-training/gsp321/wp-k8s .                              
->cd ~/wp-k8s                                             
->edit wp-env.yaml                         

Replace username_goes_here and password_goes_here to wp_user and stormwind_rules.                                
Save the changes and exit                               
  
Navigation menu->Kuberenetes Engine->Clusters                                       
Click connect button ->Copy the command line->Run in Cloud Shell                                            
OR
Run the folllowing command                                               
->gcloud container clusters get-credentials griffin-dev --zone=us-east1                                        

->kubectl apply -f wp-env.yaml                                                     

->gcloud iam service-accounts keys create key.json \                                      
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com                            
kubectl create secret generic cloudsql-instance-credentials \                               
    --from-file key.json                                            

Check my progress                                      

# Task 7: Create a WordPress deployment                                         

Cloud Shell                                                     
->cd ~/wp-k8s                                            
->edit wp-deployment.yaml                                    
                             
Replace YOUR_SQL_INSTANCE with griffin-dev-db’s Instance connection name.     
Save the changes and exit                         

For griffin-dev-db's Instance                                                    
[Navigation menu->SQL->Click on griffin-dev-db->Copy Connection Name]                                             

Esc Key                              

->:wq                                        

->kubectl create -f wp-deployment.yaml                                     
->kubectl create -f wp-service.yaml                                 

Check my Progress                               

Once the Load Balancer is created, you can visit the site and ensure you see the WordPress site installer                               

[Navigation menu->Kuberenetes Engine->Services & Ingress-> External endpoints]                         
 
# Task 8: Enable monitoring                                                               

Navigation Menu > Monitoring->Uptime Checks->Create Uptime Checks                                                      
Title : WordPress Uptime                                                     
Protocol : HTTP                                                       
Resource Type : URL                                                            
Hostname : YOUR-WORDPRESS_ENDPOINT [will get from above step after going to external endpoint link - url]                                          
Path- /                                                                                              
Next                                      
 
Test                                  

Check my Progress                                   
         
# Task 9: Provide access for an additional engineer                                      
Navigation Menu > IAM & Admin > IAM ->Click ADD                                

New members : Username2 from Console                           
Select Role : Project -> Editor                         
Save                                    

Check my Progress

# Congratualtions! you have successfully completed this challenge.
