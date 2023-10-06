	1. POD: 
	• Create a nginx pod
      ``````
      	  kubectl run nginx --image=nginx
      ``````
    
	
	• Generate POD Manifest YAML File (-o yaml). Don't create it (--dry-run)
	kubectl run nginx --image=nginx --dry-run=client -o yaml
	
	• Replace in the pod
	Kubectl replace --force -f arquivo.yaml 
	
	2. Deployment
	• Create deployment
	kubectl create deployment nginx --image=nginx
	
	• Generate Deployment YAML file (-o yaml). Don't create it (--dry-run)
	kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
	
	• Generate Deployment with 4 Replicas
	kubectl create deployment nginx --image=nginx --replicas=4
		○ You can also scale a deployment using: kubectl scale
		kubectl scale deployment nginx --replicas=4

	• Another way to do this is to save the YAML defininito to a file and modify
	kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml (You can then realize update the YAML)
	
	3. Service
	• Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379 (redis pod is created)
	kubectl expose redis pod --port=6379 --name redis-service 
	
	• Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes (nginx pod is created)
	kubectl expose nginx --type=NodePort --port=80 --name=nginx-service
	
	• Create a POD named nginx and service named nginx-serivice of type ClusterIp to expose ports 8080
kubectl run nginx-srv --image=nginx --expose=true --port=8080