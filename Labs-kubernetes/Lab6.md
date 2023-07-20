# Services
## ClusterIP

	apiVersion: v1
	kind: Service
	metadata:
	  name: svc-nginx-clusterip
	spec:
	  type: ClusterIP
	  ports:
	    - targetport: 80
	      port: 80
	  selector:
	     app: nginx
	     type: front-end

## NodePort

	apiVersion: v1
	kind: Service
	metadata:
	  name: svc-nginx-nodeport
	spec:
	  type: Nodeport
	  ports:
	    - targetport: 80
	      port: 80
	      nodePort: 30008
	  selector:
	    app: nginx
	    type: front-end

## LoadBalancer
	
	apiVersion: v1
	kind: Service
	metadata:
	  name: svc-nginx-nodeport
	spec:
	  type: Nodeport
	  ports:
	    - targetport: 80
	      port: 80
	      nodePort: 30008
	  selector:
	    app: nginx
	    type: front-end
