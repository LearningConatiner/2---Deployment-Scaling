## Deployment

+ Old method: pods/services/replication controllers separately.

+ New method: pods and replica templates defined in deployment definition.

#### yaml

No need of label

	apiVersion: v1
	kind: Deployment
	metadata:
	  name: <name>
	spec:
	  replicas: <1 - n>
	  template:
	    <from metadata to spec of pod definition, labels are needed here to associate pods to service>

## Scaling

* Horizontal scaling -  manual

		kubectl scale deployment <deployment name> --replicas <n> --namespace <namespace name>

* Auto scaling (based on load)
	
	* CPU based autoscaling
		* Pre-reqs
			1. _Heapster_ must run on cluster.				
				Heapster -> Container metric collection framework. It runs as a pod in your cluster and auto-discovers resources and reports data via syncs such as logs, InfluxDB, and more.			
			2. _Pods_ must _request resources._				
				Resource requests are requests for a certain amount of CPU or memory from the cluster. 
				
				In deployment.yaml, 
				
				* for the replicas, set a max number.
				
				* for the pod, add resource request -
							
						resources:
						  requests:
						    cpu: "100m"

**********************

1. Apply deployment changes
2. Enable auto scaling

			kubectl autoscale deployment <deployment name> --min <1> --max <10> --cpu-percent 70 --namespace <namespace name>
	
	or
	
		   kubectl hpa deployment <deployment name> --min <1> --max <10> --cpu-percent 70 --namespace <namespace name>
		   
Kubernetes will increase the number of replicas when the average CPU usage across the pod is greater or equal to 70 percent. Conversely, Kubernetes will decrease the number of replicas when the average CPU usage across the pod is less than 70 percent.

3. Unfortunately, the autoscale command can only be invoked once because it does not support updating an existing resource. We can overcome this error with kubectl edit. 

Kubectl edit opens up the specified resource in your editor, then applies the changes. It's the same as editing files yourself and calling kubectl apply.

	kubectl edit hpa <deployment> --namespace <namespace number>
	# set min - 5, max - 5 and cpu percent 10
		
		kubectl edit hpa example-app-tier --namespace ns1
		horizontalpodautoscaler "example-app-tier" edited
		
		$ watch -n 1 kubectl get deployments --namespace ns1
		Let's experiment by increasing the minimum pods. 

		kubectl get hpa <deployment> --namespace <namespace number>
		
Now you can use __watch__ command to watch the deployment auto-scale. 

**Heapster**

		kubectl get --all-namespaces services
		
		NAMESPACE     NAME                   CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
		default       kubernetes             10.7.240.1     <none>        443/TCP         1h
		kube-system   default-http-backend   10.7.251.127   <nodes>       80:31938/TCP    1h
		kube-system   heapster            10.7.241.193   <none>        80/TCP          1h
		kube-system   kube-dns               10.7.240.10    <none>        53/UDP,53/TCP   1h
		kube-system   kubernetes-dashboard   10.7.242.164   <none>        80/TCP          1h
		
		kubectl get --all-namespaces pods
		
		NAMESPACE     NAME                                           READY     STATUS    RESTARTS   AGE
		default       nginx-ex                                       1/1       Running   0          1h
		kube-system   fluentd-gcp-v2.0-741bv                         1/1       Running   0          1h
		kube-system   fluentd-gcp-v2.0-vnbm8                         1/1       Running   0          1h
		kube-system   fluentd-gcp-v2.0-zkp8q                         1/1       Running   0          1h
		kube-system   heapster-v1.3.0-1288166888-d4ll6               2/2       Running   0          1h
		kube-system   kube-dns-806549836-qx0nf                       3/3       Running   0          1h
		kube-system   kube-dns-806549836-v38xv                       3/3       Running   0          1h
		kube-system   kube-dns-autoscaler-2528518105-qd26n           1/1       Running   0          1h
		kube-system   kube-proxy-gke-pv-default-pool-caa55653-4xs7   1/1       Running   0          1h
		kube-system   kube-proxy-gke-pv-default-pool-caa55653-6v9q   1/1       Running   0          1h
		kube-system   kube-proxy-gke-pv-default-pool-caa55653-xn80   1/1       Running   0          1h
		kube-system   kubernetes-dashboard-2917854236-zxjv7          1/1       Running   0          1h
		kube-system   l7-default-backend-1044750973-dk2l8            1/1       Running   0          1h
		
		
### Useful resource monitoring commands
#### Top

kubectl top nodes

kubectl top pods --all-namespaces

#### watch

watch command -> to continously watch

