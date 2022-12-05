# CKAD_prepare

To prepare, you can try below githib/links
https://github.com/bmuschko/ckad-prep
https://github.com/aireddy73/Kubernetes/tree/main/CKAD%20Exercies
https://github.com/bmuschko/CKAD-exercises
https://github.com/dguyhasnoname/CKAD-TheHardWay
https://github.com/dgkanatsios/CKAD-exercises

Command to create resources `kubectl create --help`

Get fields `kubectl explain pods.spec`

use the alias `alias k=kubectl`

Setting context & Namespace `kubectl config set-context --current --namespace=app-spac`

ShortCut
instead of namepaces use ns `kubectl get ns`
instead of using pods use po `kubectl get po`
instead of deployment use deploy `kubectl get deploy`
similarily
persistentvolumeclaim=pvc
persistentvolume=pv

Delete any K8s object `kubectl delete pod nginx --grace-period=0 --force`

to schedule the pod on specific node called production node `kubectl taint nodes production-node app=red:NoSchedule`

Frequently used commands
set default namespace `kubectl config set-context --current --namespace=ckad-prep`
Rolling update `kubectl set image deployment/backend www=nginx:1.1.2`
Checking history of deployment `kubectl rollout history deployment/frontend`
rollback to previous version `kubectl rollout undo deployment/backend`
rollback to specific version `kubectl rollout undo deployment/backend --to-revision=2`
watch status `kubectl rollout status deployment/backend`
Copy /tmp/foo_dir local directory to /tmp/bar_dir in a remote pod in the current namespace `kubectl cp /tmp/foo_dir my-pod:/tmp/bar_dir`
Copy /tmp/foo local file to /tmp/bar in a remote pod in a specific container `kubectl cp /tmp/foo my-pod:/tmp/bar -c my-container`
Copy /tmp/foo local file to /tmp/bar in a remote pod in namespace my-namespace `kubectl cp /tmp/foo my-namespace/my-pod:/tmp/bar`
Copy /tmp/foo from a remote pod to /tmp/bar locally `kubectl cp my-namespace/my-pod:/tmp/foo /tmp/bar`

listen on local port 5000 and forward to port 5000 on Service backend `kubectl port-forward svc/my-service 5000`
listen on local port 5000 and forward to Service target port with name my-service-port `kubectl port-forward svc/my-service 5000:my-service-port`
listen on local port 5000 and forward to port 6000 on a Pod created by my-deployment`kubectl port-forward deploy/my-deployment 5000:6000`
run command in first Pod and first container in Deployment (single- or multi-container cases) `kubectl exec deploy/my-deployment -- ls`

kubectl cordon my-node                                                Mark my-node as unschedulable
kubectl drain my-node                                                 Drain my-node in preparation for maintenance
kubectl uncordon my-node                                              Mark my-node as schedulable
kubectl top node my-node                                              Show metrics for a given node
kubectl cluster-info                                                  Display addresses of the master and services
kubectl cluster-info dump                                             Dump current cluster state to stdout
kubectl cluster-info dump --output-directory=/path/to/cluster-state   Dump current cluster state to /path/to/cluster-state

kubectl api-resources --namespaced=true       All namespaced resources
kubectl api-resources --namespaced=false      All non-namespaced resources
kubectl api-resources -o name                 All resources with simple output (only the resource name)
kubectl api-resources -o wide                 All resources with expanded (aka "wide") output
kubectl api-resources --verbs=list,get        All resources that support the "list" and "get" request verbs
kubectl api-resources --api-group=extensions  All resources in the "extensions" API group

Run a command "env" `kubectl run nginx --image=nginx -- env`

create resource quota `kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml`

setting a pod image `kubectl set image pod/nginx nginx=nginx:1.7.1`

getting image version ` kubectl get po nginx -o jsonpath='{.spec.containers[].image}'`

run a wget in image `k run busybox123 --image=busybox -- wget -O- 10.42.0.171:80`

get ip of the pod `k get po myapp-6ffd94cbc8-88vpm -o jsonpath='{.status.podIP}'`

set environment variable `kubectl run nginx --image=nginx --restart=Never --env=var1=val1`

# Labels

Show Labels `kubectl get pods --show-labels`

Show Label by values `kubectl get po -l 'team in (shiny, legacy)',env=prod --show-label`

Show 3 lines around a text `kubectl get pods -o yaml | grep -C 3 'annotations'`

to remove all labels `kubectl label pod backend env-`

change a label `kubectl label po nginx2 app=v2 --overwrite`

Add a new label to pod having app=v1 or v2 `kubectl label pod -l 'app in (v1,v2) tier=web`

remove label from pod having app as label `kubectl label po -l app app-`

Run a pod specifically on a node
`kubectl label nodes node1 accelerator=nvidia-tesla-p100`
Use Nodeselector property to set node affinity
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-tesla-p100            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

Annotate pod nginx1 with description='my description'
`kubectl annotate po/nginx description='my description'`

Annotate pod nginx1, nginx2 and nginx3 with description='my description'
`kubectl annotate po/nginx{1..3} desciption='my description'`

check annotation
`kubectl annotate po/nginx1 --list`

Remove an annotation
`kubectl annotate po/nginx{1..3} description-`



Assigning Labels
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: pod1
	labels:
		tier: backend
 		env: prod
 		app: miracle
spec:
-----
```
`kubectl label pod nginx tier=backend env=prod app=miracle `
`kubectl get pods --show-labels`
NAME ... LABELS
pod1 ... tier=backend,env=prod,app=miracle

Tier is "frontend" AND is "development" environment `kubectl get pods -l tier=frontend,env=dev --show-labels`
NAME ... LABELS
pod2 ... app=crawler,env=dev,tier=frontend

Has the label with key "version" `kubectl get pods -l version --show-labels`
NAME ... LABELS
pod3 ... app=crawler,env=staging,version=v2.1

Tier is in set "frontend" or "backend" AND is "development" environment`kubectl get pods -l 'tier in (frontend,backend),env=dev' --show-labels`
NAME ... LABELS
pod2 ... app=crawler,env=dev,tier=frontend

Selecting Resources in YAML
```yaml
Equality-based
apiVersion: v1
kind: Service
metadata:
	name: app-service
 ...
spec:
 ...
 	selector:
 		tier: frontend
 		env: dev
```

Equality- and set-based
```yaml
apiVersion: batch/v1
kind: Job
metadata:
	name: my-job
spec:
	...
	selector:
		matchLabels:
 			version: v2.1
 		matchExpressions:
 			- {key: tier, operator: In,↵
 				values: [frontend,backend]}
```


# Annotations
Purpose of Annotations - Descriptive metadata without the ability to be queryable
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: my-pod
 	annotations:
 		commit: 866a8dc
 		author: 'Benjamin Muschko'
 	branch: 'bm/bugfix'
spec:
....
```
`kubectl annotate pod nginx `
commit='866a8dc' author='Benjamin Muschko' branch='bm/bugfix'

`kubectl describe pods my-pod`
Name: my-pod
Namespace: default
...
Annotations: author: Benjamin Muschko
 branch: bm/bugfix
 commit: 866a8dc
...


# Deployment
create deployment `kubectl create deployment myapp --image=nginx --port=80`

`kubectl create deployment my-deploy --image=nginx --replicas=3 --dry-run=client -o yaml> deploy.yaml`

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-deploy
   name: my-deploy
spec:
   replicas: 3
   selector:
     matchLabels:
 		app: my-deploy
 	template:
 	  metadata:
 		labels:
 	      app: my-deploy
 	  spec:
 		containers:
 		- image: nginx
 		  name: nginx
```

change image `kubectl set image deployment/deploy nginx=nginx:latest`

create deployment exposing nodeport on port=8080 `kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080`


Check current deployment replicas
`kubectl get deployments my-deploy`
NAME READY UP-TO-DATE AVAILABLE AGE
my-deploy 2 2 2 9h

Scaling from 2 to 4 replicas
`kubectl scale deployment my-deploy --replicas=4`
deployment.extensions/my-deploy scaled

Check the changed deployment replicas
`kubectl get deployment my-deploy`
NAME READY UP-TO-DATE AVAILABLE AGE
my-deploy 4 4 4 9h

# Create Horizontal Pod Autoscaler
 Maintain average CPU utilization across all Pods of 70%
`kubectl autoscale deployments my-deploy --cpu-percent=70  --min=1 --max=10`
horizontalpodautoscaler.autoscaling/my-deploy autoscaled

Check the current status of autoscaler
`kubectl get hpa my-deploy`
NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
my-deploy Deployment/my-deploy 0%/70% 1 10 4 23s

pause deployment
`kubcel rollout pause deploy nginx`

resume deployment
`kubectl rollout resume deploy/nginx`



# Pod
pod creation and running a command `kubectl run tmp --image=busybox --restart=Never -it --rm -- wget -O- 10.109.232.76:80`

kubectl describe pods | grep -C 10 "author=John Doe"

Run a pod `kubectl run nginx --image=nginx --dry-run=client -o yaml > ngix-pod.yaml`

```yaml
Configuring Env Variables
apiVersion: v1
kind: Pod
metadata:
	name: spring-boot-app
spec:
	containers:
 	- image: busybox
 		name: spring-boot-app
 		env:
		- name: SPRING_PROFILES_ACTIVE
 		  value: production

Command and Arguments
apiVersion: v1
kind: Pod
metadata:
	name: spring-boot-app
spec:
	containers:
 	- image: busybox
 		name: spring-boot-app
 		args:
 		- /bin/sh
 		- -c
 		- echo "started app"
```

# Job
Increment a counter and render its value on the terminal
`kubectl create job counter --image=nginx -- /bin/sh -c 'counter=0; while [ $counter -lt 3 ]; do  counter=$((counter+1)); echo "$counter"; sleep 3; done;'`
job.batch/counter created

Creating a Job (declarative)
```yaml
apiVersion: batch/v1
kind: Job
metadata:
 	name: counter
spec:
 	completions: 1
 	parallelism: 1
 	backoffLimit: 6
 	template:
 		spec:
 			restartPolicy: OnFailure
 			containers:
 			- args:
 				- /bin/sh
 				- -c
 				- ...
 		image: nginx
 		name: counter
```

 List all jobs
`kubectl get jobs`
NAME DESIRED SUCCESSFUL AGE
counter 1 1 3m

Identify correlating Pods
`kubectl get pods`
NAME READY STATUS RESTARTS AGE
counter-924lc 0/1 Completed 0 22m

Get the logs of the Pod
`kubectl logs counter-924lc`
1
2
3

spec.template.spec.restartPolicy: OnFailure
Same pod will be restarted again

spec.template.spec.restartPolicy: Never
New pod will restart



spec.completions: x
spec.parallelism: y

Type Completion criteria
Non-parallel --Complete as soon as its Pod terminates successfully
Parallel with fixed completion count -- Complete when specified number of tasks finish successfully
Parallel with a work queue -- Once at least one Pod has terminated with success and all Pods are terminated


# Logs Access
Accessing Logs `kubcetl logs busybox`
Dump logs of container 1 `kubectl logs busybox --container=c1`
Log into container 2 `kubectl exec busybox -it --container=c2 -- /bin/sh`

# Update 
to view history of rollout  `kubectl rollout history deploy`
 
changes in that rollout `kubectl rollout history deploy --revision=5`

rollback to older version `kubectl rollout undo deployment/deploy --to-revision=4`


# Cronjob
cronjob creation `kubectl create cronjob current-date --schedule="* * * * *" --image=nginx -- /bin/sh -c 'echo "Current date: $(date)"'`

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
 	name: counter
spec:
 	schedule: "*/1 * * * *"
 		jobTemplate:
 			spec:
 				template:
 					spec:
 						restartPolicy: Never
 						containers:
 						- args:
 							- /bin/sh
 							- -c
 							- ...
 							image: nginx
 							name: counter
``` 							

# Service 
create service with cluster ip `kubectl create service clusterip myapp --tcp=80:80`

create a pod ad expose with a service `kubectl run nginx --image=nginx --port=80 --expose`

```yaml
apiVersion: v1
kind: Service
metadata:
 	name: nginx
spec:
 	selector:
 		tier: frontend --- This is how service maps pod to redirect request
 	ports:
 		- port: 3000
 			protocol: TCP
 			targetPort: 80
 	type: ClusterIP
```

Only reachable from within the cluster
`kubectl get service nginx`
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
nginx ClusterIP 10.105.201.83 <none> 80/TCP 3h

Accessible from outside of the cluster
`kubectl get service nginx`
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
nginx NodePort 10.105.201.83 <none> 80:30184/TCP 3h


# Network
view network policy `kubectl get networkpolicy`


```
yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
 	name: my-network-policy
spec:
 	podSelector:
 		matchLabels:
 			tier: frontend ---label selection for pods
 	policyTypes:
 	- Ingress   -- inbound and outbound traffic
 	- Egress
 	ingress:
 	- from:     -- who can connect to pod?     
 		...
 	egress:
 	- to:      
 ...          -- where can pod connect to?

```

Denying all access and allowing access as needed
```
yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
 	name: default-deny
spec:
 	podSelector: {}  --applies to all pods
 	policyTypes:
 	- Ingress -- inbound and outbound blocked
 	- Egress
```

Allow traffic from pod with label
```
yaml
...
ingress:
	- from:
 		- podSelector:
 				matchLabels:
 					tier: backend -- allow inbound traffic from pod having label=backend
...

```

Restrict access to ports
```yaml
...
ingress:
	- from:
 		- podSelector:
 				matchLabels:
 					tier: backend
 		ports:
 			- protocol: TCP   -- allow access from 5432 port
 				port: 5432
...
```

# Commands

if [ ! -d ~/tmp ]; then mkdir -p ~/tmp; fi; while true;↵
do echo $(date) >> ~/tmp/date.txt; sleep 5; done;

# Connecting Pod
Connecting to a running pod `kubectl exec nginx --it -- /bin/sh`

# Configmap
Literal values `kubectl create configmap db-config --from-literal=db=staging`
Single file with environment variables `kubectl create configmap db-config --from-env-file=config.env`
File or directory `kubectl create configmap db-config --from-file=config.txt`

```
yaml
Creating ConfigMaps (declarative)
apiVersion: v1
data:
	db: staging
kind: ConfigMap
metadata:
	name: db-config

Configmap env var in pod
apiVersion: v1
kind: Pod
metadata:
	name: backend
spec:
	containers:
 	- image: nginx
 	name: backend
 	envFrom:
 		- configMapRef:
 			name: db-config
```
```
yaml
ConfigMap in pod as volume 			
apiVersion: v1
kind: Pod
metadata:
	name: backend
spec:
	containers:
 		- name: backend
 		  image: nginx
 		  volumeMounts:
 			- name: config-volume
 			mountPath: /etc/config
 	volumes:
 		- name: config-volume
 		  configMap:
 			name: db-config
```

# Secrets (imperative)
Literal values `kubectl create secret generic db-creds --from-literal=pwd=s3cre!`
File containing environment variables `kubectl create secret generic db-creds --from-env-file=secret.env`
SSH key file `$ kubectl create secret generic db-creds --from-file=ssh-privatekey=~/.ssh/id_rsa`

```
yaml
Creating Secrets (declarative)
apiVersion: v1
kind: Secret
metadata:
	name: mysecret
type: Opaque
data:
	pwd: czNjcmUh

Secret in Pod as Volume
apiVersion: v1
kind: Pod
metadata:
	name: backend
spec:
	containers:
 		- name: backend
 	  		image: nginx
      		volumeMounts:
 	  		- name: secret-volume
 			  mountPath: /etc/secret
 	volumes:
 		- name: secret-volume
 		  secret:
 			secretName: mysecret	
```

# Readiness and Liveness Probe
Defining a Readiness Probe
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: web-app
spec:
 	containers:
 	- name: web-app
 	  image: eshop:4.6.3
 	  readinessProbe:
 		httpGet:
 			path: /
 			port: 8080
 		initialDelaySeconds: 5
 		periodSeconds: 2
```

Defining a Liveness Probe
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: web-app
spec:
 	containers:
 	- name: web-app
 	  image: eshop:4.6.3
 	  livenessProbe:
 		exec:
 			command:
 			- cat
 			- /tmp/healthy
 		initialDelaySeconds: 10
 		periodSeconds: 5
```
```yaml
Defining a Startup Probe
apiVersion: v1
kind: Pod
metadata:
	name: startup-pod
spec:
 	containers:
 	- image: httpd:2.4.46
 	  name: http-server
 	  startupProbe:
 		tcpSocket:
 			port: 80
 		initialDelaySeconds: 3
 		periodSeconds: 15
```

# Defining a Security Context
```
yaml
apiVersion: v1
kind: Pod
metadata:
	name: secured-pod
spec:
	securityContext:
		runAsUser: 1000
 	containers:
 	- securityContext:
 		runAsGroup: 3000

Creating a Resource Quota
apiVersion: v1
kind: ResourceQuota
metadata:
	name: app
spec:
 	hard:
 		pods: "2"
 		requests.cpu: "2"
 		requests.memory: 500m
```

Defining Container Constraints
```
yaml
apiVersion: v1
kind: Pod
metadata:
	name: mypod
spec:
	containers:
 	- image: nginx
 	  name: mypod
      resources:
  		requests:
 			cpu: "0.5"
 			memory: "200m"
```

Declaring Service Accounts
```
yaml
apiVersion: v1
kind: Pod
metadata:
 	name: app
spec:
 	serviceAccountName: myserviceaccount
```

Defining an Init Container
```
yaml
apiVersion: v1
kind: Pod
metadata:
	name: multi-container
spec:
	initContainers:
 	- image: init:3.2.1
 	  name: app-initializer
	containers:
 	- image: nginx
 	  name: web-server
```

Defining Multiple Containers
```
yaml
apiVersion: v1
kind: Pod
metadata:
	name: multi-container
spec:
 	containers:
 	- image: nginx
 		name: container1
 	- image: nginx
 		name: container2
```

# Design Patterns
Ambassador Pattern
Proxy for main application container

# Kubernetes Concepts
Q - How does Kubernetes know if a container is up and running?
A - Probes can detect and correct failures

HTTP GET Request httpGet
Sends an HTTP GET request to an endpoint exposed by the application. A HTTP response code in the range of 200 and 399 indicates success. Any other response code is regarded as an error.

TCP socket connection tcpSocket
Tries to open a TCP socket connection to a port. If the connection could be established, the probing attempt was successful. The inability to connect is accounted for as an error.

Q Common Pod Error Statuses
A ImagePullBackOff or ErrImagePull
Image could not be pulled from registry Check correct image name, check that image name exists in registry, verify network access from node to registry, ensure proper

CrashLoopBackOff Application or command run in container crashes Check command executed in container, ensure that image can properly execute (e.g. by creating
a container with Docker)

CreateContainerConfigError ConfigMap or Secret references by container cannot be found Check correct name of the configuration object, verify the existence of the configuration object in the namespace

Q Can you foresee potential issues with a rolling deployment?
A rolling deployment ensures zero downtime which has the side effect of having two different versions of a container running at the same time. This can become an issue if you introduce backward-incompatible changes to your public API. A client might hit either the old or new service API.

Q How do you configure a update process that first kills all existing containers with the current version before it starts containers with the new version?
A You can configure the deployment use the Recreate strategy. This strategy first kills all existing containers for the deployment running the current version before starting containers running the new version.

Check initial deployment revisions `kubectl rollout history deployments my-deploy`
deployment.extensions/my-deploy
REVISION CHANGE-CAUSE
1 <none>

Make a change to the deployment `kubectl edit deployments my-deploy`
# Revision history indicates changed version

`kubectl rollout history deployments my-deploy`
deployment.extensions/my-deploy
REVISION CHANGE-CAUSE
1 <none>
2 <none>

Rendering Revision Details
`kubectl rollout history deployments my-deploy --revision=2`
```yaml
deployment.extensions/my-deploy with revision #2
Pod Template:
 	Labels: app=my-deploy
pod-template-hash=1365642048
 	Containers:
 	  nginx:
 		Image: nginx:latest
 		Port: <none>
 		Host Port: <none>
 		Environment: <none>
 		Mounts: <none>
 		Volumes: <none>
```

# Volumes

emptyDir -- Empty directory in Pod. Only persisted for the lifespan of a Pod.
hostPath -- File or directory from the host node’s filesystem into your Pod.
configMap, secret -- Provides a way to inject configuration data and secrets into Pods.
nfs      -- An existing NFS (Network File System) share to be mounted into your Pod. Preserves data after Pod restart.
Cloud provider solutions -- Provider-specific implementation for AWS, GCE or Azure

```yaml
apiVersion: v1
kind: Pod
metadata:
 	name: my-container
spec:
 	volumes:
 	- name: logs-volume
 		emptyDir: {}
 containers:
 	- image: nginx
 		name: my-container
 		volumeMounts:
 		- mountPath: /var/logs
 			name: logs-volume
```

Creating volume
```yaml
apiVersion: v1
kind: Pod
metadata:
 	name: my-container
spec:
 	volumes:
 	- name: logs-volume
 		emptyDir: {}
 	containers:
 	- image: nginx
 		name: my-container
 		volumeMounts:
 		- mountPath: /var/logs
 			name: logs-volume
```

Creating a persistent volume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 	name: pv
spec:
 	capacity:
 		storage: 512m
 	accessModes:  			-- how many nodes can access volumes
 		- ReadWriteOnce
 	storageClassName: shared
 	hostPath:
 		path: /data/config
```

Creating a claim
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
 	name: pvc
spec:
 	accessModes:
 		- ReadWriteMany
 	resources:
 		requests:
 			storage: 256m
 	storageClassName: shared
 ```

 Mounting a claim
 ```yaml
apiVersion: v1
kind: Pod
metadata:
 	creationTimestamp: null
 	name: app
spec:
 	volumes:
 	- name: configpvc
 		persistentVolumeClaim:
 			claimName: pvc
 	containers:
 	- image: nginx
 		name: app
 		volumeMounts:
 			- mountPath: "/data/app/config"
 				name: configpvc
 ```


