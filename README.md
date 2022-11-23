# CKAD_prepare

To prepare, you can try below githib/links
https://github.com/bmuschko/ckad-prep


kubectl run mypod --image=nginx:2.3.5 --restart=Never --port=80 --namespace=ckad-prep

Command to create resources
kubectl create --help

Get fields
kubectl explain pods.spec

use the alias
alias k=kubectl

Setting context & Namespace
kubectl config set-context --current --namespace=app-spac

ShortCut
instead of namepaces use ns
kubectl get ns
instead of using pods use po
kubectl get po
instead of deployment use deploy
kubectl get deploy
similarily
persistentvolumeclaim=pvc
persistentvolume=pv

Delete any K8s object
kubectl delete pod nginx --grace-period=0 --force

kubectl taint nodes production-node app=red:NoSchedule
to schedule the pod on specific node called production node

kubectl get pods --show-labels

kubectl get po -l 'team in (shiny, legacy)',env=prod --show-label

kubectl get pods -o yaml | grep -C 3 'annotations'

kubectl label pod backend env-
to remove all labels

change image
kubectl set image deployment/deploy nginx=nginx:latest

to view history of rollout
kubectl rollout history deploy

changes in that rollout
kubectl rollout history deploy --revision=5

rollback to older version
kubectl rollout undo deployment/deploy --to-revision=4

cronjob creation 
kubectl create cronjob current-date --schedule="* * * * *" --image=nginx -- /bin/sh -c 'echo "Current date: $(date)"'

create service with cluster ip
kubectl create service clusterip myapp --tcp=80:80

create deployment
`kubectl create deployment myapp --image=nginx --port=80`

kubectl create deployment my-deploy --image=nginx --replicas=3 --dry-run=client -o yaml> deploy.yaml

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



pod creation and running a command
kubectl run tmp --image=busybox --restart=Never -it --rm -- wget -O- 10.109.232.76:80	

create deployment exposing nodeport on port=8080
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080

view network policy
kubectl get networkpolicy

Commands
if [ ! -d ~/tmp ]; then mkdir -p ~/tmp; fi; while true;↵
do echo $(date) >> ~/tmp/date.txt; sleep 5; done;

kubectl describe pods | grep -C 10 "author=John Doe"

Run a pod
kubectl run nginx --image=nginx --dry-run=client↵
 -o yaml > ngix-pod.yaml

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

Accessing Logs
kubcetl logs busybox

Connecting to a running pod
kubectl exec nginx --it -- /bin/sh

Creating configmap
# Literal values
kubectl create configmap db-config --from-literal=db=staging
# Single file with environment variables
kubectl create configmap db-config --from-env-file=config.env
# File or directory
kubectl create configmap db-config --from-file=config.txt 		

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


Creating Secrets (imperative)
# Literal values
$ kubectl create secret generic db-creds --from-literal=pwd=s3cre!
# File containing environment variables
$ kubectl create secret generic db-creds --from-env-file=secret.env
# SSH key file
$ kubectl create secret generic db-creds --from-file=ssh-privatekey=~/.ssh/id_rsa

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

Defining a Security Context
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

Defining Container Constraints
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

Declaring Service Accounts
apiVersion: v1
kind: Pod
metadata:
 	name: app
spec:
 	serviceAccountName: myserviceaccount

Defining an Init Container
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

Defining Multiple Containers
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

Targeting Different Containers
Dump logs of container 1
kubectl logs busybox --container=c1 
Log into container 2
kubectl exec busybox -it --container=c2 -- /bin/sh

Ambassador Pattern
Proxy for main application container

Container Health
“How does Kubernetes know if a container is up and running?”
Probes can detect and correct failures

Custom Command exec.command
Executes a command inside of the container e.g. a cat command and checks its exit code. Kubernetes considers an zero exit code to be successful. A non-zero exit code indicates an error.

HTTP GET Request httpGet
Sends an HTTP GET request to an endpoint exposed by the application. A HTTP response code in the range of 200 and 399 indicates success. Any other response code is regarded as an error.

TCP socket connection tcpSocket
Tries to open a TCP socket connection to a port. If the connection could be established, the probing attempt was successful. The inability to connect is accounted for as an error.

Defining a Readiness Probe
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

Defining a Liveness Probe
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

Common Pod Error Statuses
ImagePullBackOff or ErrImagePull
Image could not be pulled from registry Check correct image name, check that image name exists in registry, verify network access from node to registry, ensure proper

CrashLoopBackOff Application or command run in container crashes Check command executed in container, ensure that image can properly execute (e.g. by creating
a container with Docker)

CreateContainerConfigError ConfigMap or Secret references by container cannot be found Check correct name of the configuration object, verify the existence of the configuration object in the namespace

Assigning Labels
apiVersion: v1
kind: Pod
metadata:
	name: pod1
	labels:
		tier: backend
 		env: prod
 		app: miracle
spec:
---
$ kubectl label pod nginx tier=backend 
env=prod app=miracle 
$ kubectl get pods --show-labels
NAME ... LABELS
pod1 ... tier=backend,env=prod,app=miracle

# Tier is "frontend" AND is "development" environment
$ kubectl get pods -l tier=frontend,env=dev --show-labels
NAME ... LABELS
pod2 ... app=crawler,env=dev,tier=frontend
# Has the label with key "version"
$ kubectl get pods -l version --show-labels
NAME ... LABELS
pod3 ... app=crawler,env=staging,version=v2.1
# Tier is in set "frontend" or "backend" AND is "development" environment
$ kubectl get pods -l 'tier in (frontend,backend),env=dev' --show-labels
NAME ... LABELS
pod2 ... app=crawler,env=dev,tier=frontend

# Selecting Resources in YAML
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

Equality- and set-based
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


# Purpose of Annotations		
Descriptive metadata without the ability to be queryable

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

$ kubectl annotate pod nginx 
commit='866a8dc' author='Benjamin Muschko' branch='bm/bugfix'
$ kubectl describe pods my-pod
Name: my-pod
Namespace: default
...
Annotations: author: Benjamin Muschko
 branch: bm/bugfix
 commit: 866a8dc
...



Can you foresee potential issues with a rolling deployment?

A rolling deployment ensures zero downtime which has the side effect of having two different versions of a container running at the same time. This can become an issue if you introduce backward-incompatible changes to your public API. A client might hit either the old or new service API.

How do you configure a update process that first kills all existing containers with the current version before it starts containers with the new version?

You can configure the deployment use the Recreate strategy. This strategy first kills all existing containers for the deployment running the current version before starting containers running the new version.