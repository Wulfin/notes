# Kubernetes' notes 

### Kubernetes

#### What is Kubernetes?

- What if you could just define "This process should have 6 copies using X amount of resources." and have the *2..N* computers working as a single entity to fulfill your request? That's just one thing Kubernetes makes possible
- In essence, Kubernetes is the sum of all the bash scripts and best practices that most system administrators would cobble together over time, presented as a single system behind a declarative set of APIs
- Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications. It groups containers that make up an application into logical units for easy management and discovery
- The main responsibility of an orchestration system:
	+ Starting and stopping of containers
 	+ Networking between containers
 	+ Health monitoring
 	+ We want the system to keep the application automatically healthy
- [Kubernetes comic](https://cloud.google.com/kubernetes-engine/kubernetes-comic/)
- When managing Docker images, Kubernetes also makes applications portable. Once they are developed with a containerized architecture using Kubernetes, they can be deployed anywhere – public cloud, hybrid, on-prem – without any change to the underlying code.

#### Kubernetes cluster with k3d

- A cluster is a group of machines, nodes, that work together
- "Server nodes" are nodes with control-plane
- "Agent nodes" are nodes without that role

![With k3d](https://github.com/FAJOUIAnas/Moriono/assets/93566369/68953fac-78b5-4843-b4b6-50d7f3734dce "without k3d")
- A cluster with k3d:

![Without k3d](https://github.com/FAJOUIAnas/Moriono/assets/93566369/8d4592a5-4d6d-4123-a2af-7daa4aebac5b "with k3d")

#### kubectl

- The Kubernetes command-line tool
- Allows to interact with the cluster
- Read kubeconfig and use the information to connect to the cluster
- The contents include certificates, passwords and the address in which the cluster API

#### Useful kubernetes commands
- kubectl port-forward hashresponse-dep-57bcc888d7-dj5vk 3003:3000
- kubectl get <object>s
- kubectl cluster-info
- kubectl explain <object>
- kubectl create deployment hashgenerator-dep --image=jakousa/dwk-app1
- kubectl scale deployment/hashgenerator-dep --replicas=4
- kubectl delete <object> <name>
- kubectl describe <object> <name>

#### Concepts

- Pod
	+ An abstraction around one or more containers
	+ Container of containers
- ReplicaSets
	+ Used to tell how many replicas of a Pod you want
	+ It will delete or create Pods until the number of Pods you wanted are running
	+ ReplicaSets are managed by Deployments
- Deployment
  	+ Provides declarative updates for Pods and ReplicaSets
  	+ You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate
  	+ A Deployment resource takes care of deployment. It's a way to tell Kubernetes what container you want, how they should be running and how many of them should be running
  	+ Kubernetes take care of rolling out a new version of a deployment by using tags (e.g. hehe/image:tag), with the deployments each time we update the image we can modify and apply the new deployment yaml

- Updating images
	+ Kubernetes doesn't know if a Docker image is modified or wether a newer version is uploaded to the repository
	+ By default, Kubernetes won't pull the image if it already exists
	+ By using tags (e.g. fjwix/image:tag) with the deployments each time you update the image you can modify and apply the new deployment yaml. From the tag Kubernetes will know that the image is a new one and pulls it.
	+ When you first create a Deployment, StatefulSet, Pod, or other object that includes a Pod template, then by default the pull policy of all containers in that pod will be set to `IfNotPresent` if it is not explicitly specified. This policy causes the kubelet to skip pulling an image if it already exists.
	+ The `imagePullPolicy` for a container and the tag of the image affect when the kubelet attempts to pull (download) the specified image. Here's a list of the values you can set for imagePullPolicy and the effects these values have:
		* **`IfNotPresent`:** the image is pulled only if it is not already present locally
		* **`Always`:** every time the kubelet launches a container, the kubelet queries the container image registry to resolve the name to an image digest. If the kubelet has a container image with that exact digest cached locally, the kubelet uses its cached image; otherwise, the kubelet pulls the image with the resolved digest, and uses that image to launch the container.
		* **`Never`:** the kubelet does not try fetching the image. If the image is somehow already present locally, the kubelet attempts to start the container; otherwise, startup fails. See pre-pulled images for more details.
	+ The caching semantics of the underlying image provider make even imagePullPolicy: Always efficient, as long as the registry is reliably accessible. Your container runtime can notice that the image layers already exist on the node so that they don't need to be downloaded again.


 
#### Networking

- [Webinar: Kubernetes and Networks: Why is This So Dang Hard?](https://youtu.be/GgCA2USI5iQ?si=Pc7d4OfzgOQoFLRp)
- Service
  	+ Service resource will take care of serving the application to connections from outside of the cluster
  	+ Used to expose and access applications running within a Kubernetes cluster
  	+ Enable network communication between different parts of your application or between different applications
  	+ They provide a consistent and stable endpoint to interact with your application, regardless of how many pods or replicas are running, their IP addresses, or their current state
  	+ Types:
  		* ClusterIP
  	 		- Internal service (only accessible within the cluster)
  	   		- Useful for inter-pod communication
		* NodePort
			- Exposes the service on a static port on each node
  			- Used for external access to a service
  	  		- Not flexible and require you to assign a different port for every application
  	    		- Are not used in production but are helpful to know about
  	     		- Service.yaml :

						apiVersion: v1
						kind: Service
						metadata:
						  name: my-nodeport-service
						spec:
						  type: NodePort
						  selector:
							app: my-app # This is the app as declared in the deployment.
						  ports:
							- name: http
							  nodePort: 30080 # This is the port that is available outside. Value for nodePort can be between 30000-32767
							  protocol: TCP
							  port: 1234 # This is a port that is available to the cluster, in this case it can be ~ anything
							  targetPort: 3000 # This is the target port

		* LoadBalancer:
			- Creates an external load balancer
			- Useful in cloud environments where load balancers can be provisioned automatically
		* ExternalName:
			- Maps a service to a DNS name
			- Acts as CNAME record
- Ingress
	+ Incoming Network Access resource
  	+ Different type of resource from *Services*
  		* In OSI model, it works in layer 7 while services work on layer 4
  	 	* Can be used together: first a *LoadBalancer* and then Ingress to handle routing
	+ Similar to Nginx

#### Storage

##### Storage on Kubernetes is hard

- [Why Is Storage On Kubernetes So Hard?](https://softwareengineeringdaily.com/2019/01/11/why-is-storage-on-kubernetes-is-so-hard/)
- Kubernetes does not support storing state. Almost all production applications are stateful, i.e. require some sort of external storage.
- A Kubernetes architecture is very dynamic. Containers are being created and destroyed, depending on the load and on the specifications of the developers. Pods and containers can self-heal and replicate. They are, in essence, ephemeral.
- A persistent storage solution cannot afford this dynamic behavior. **Persistent storage cannot be bound to the rules of being dynamically created and destroyed.**
- The storage landscape for cloud native applications is not easy to understand. The [Kubernetes storage lingo](https://www.youtube.com/watch?v=uSxlgK1bCuA) can be confusing, with many terms that have intricate meanings and subtle changes.

##### Volume plugins

- Kubernetes communicate with storage using control plane interfaces
- The interfaces link Kubernetes with external storage
- The external storage solutions linked to Kubernetes are called Volume plugins
- Volume Plugins enable abstracting storage and grant storage portability

##### Storage in Native Kubernetes

- Kubernetes natively offers some solutions to manage storage:
	+ Ephemeral options
	+ Persistent storage:
		* Piersistent Volumes
		* Persistent Volume Claims
		* Storage Classes
		* StatefulSets


#### Best practices
  
- When updating anything in Kubernetes the usage of delete is actually an anti-pattern and you should use it only as the last option. As long as you don't delete the resource Kubernetes will do a rolling update, ensuring minimum (or none) downtime for the application. On the topic of anti-patterns: you should also always avoid doing anything imperatively! If your files don't tell Kubernetes and your team what the state should be and instead you run commands that edit the state you are just lowering the bus factor for your cluster and application.
- Often you (the maintainer or developer) don't have to do anything in case something goes wrong with a pod or a container. Sometimes you need to interfere, or you might have problems with your own configuration.
- Debugging tools:
  + kubectl describe
  + kubectl logs
  + kubectl delete

## Networking

### Reverse proxy
![Reverse proxy](https://github.com/FAJOUIAnas/Moriono/assets/93566369/f26f0cd1-3260-4692-b398-4daee2571abd "Diagram of a reverse proxy")
- Is an application that sits in front of back-end applications and forwards client (e.g. browser) requests to those applications
- The resources returned to the client appear as if they originated from the web server itself
- Can help increase:
	+ Scalability
 	+ Performance
  	+ Resilience
  	+ Security
- Can keep a cache of static content
- Can be used to add features such as compression or TLS encryption to the communication channel between the client and the reverse proxy