# k8s-nodejs-demo

IMPORTANT: In order to complete the demo you will need to login your docker registry and setup your kubeconfig properly to access your kubernetes cluster via kubectl.


Clone the project and build your docker image locally:  
```bash
git clone https://github.com/jmgarciac/k8s-nodejs-demo.git
cd openshift-nodejs-demo
sed -i "s/Hello World/$USER says \"Hello World\" ;-)/g" app/server.js
docker build -t ${USER}-nodejs:v1 .
docker run -p 8080:8080 -d ${USER}-nodejs:v1
```

Setup your environment  
(remember change the values to fit your environment) :  
``` bash
export NS="${USER}-demo"
export REGISTRY_URL=<MY-REGISTRY>
export PUBLIC_IP=<MY_IP>
```

Create a namespace for your application deployment  
(cluster privileges required, ask your cluster administrator):  
``` bash
kubectl create ns ${NS}
kubectl create rolebinding ${NS}-admin --clusterrole=admin --user <USER>
```

Push (upload) your local image to the Container Registry:  
``` bash
docker tag ${USER}-nodejs:v1 ${REGISTRY_URL}/${USER}-nodejs:v1
docker push ${REGISTRY_URL}/${USER}-nodejs:v1
```

Deploy your application creating a simple deployment with two replicas:
``` bash
kubectl -n ${NS} create deployment nodejs-deployment --image ${REGISTRY_URL}/${USER}-nodejs:v1 --port 8080 --replicas 2 --dry-run=client -o yaml
kubectl -n ${NS} create deployment nodejs-deployment --image ${REGISTRY_URL}/${USER}-nodejs:v1 --port 8080 --replicas 2
kubectl -n ${NS} get deployment
kubectl -n ${NS} get pods
```

Create the service needed to balance the load between the replicas:  
``` bash
kubectl -n ${NS} expose deployment nodejs-deployment --name nodejs-service --dry-run=client -o yaml
kubectl -n ${NS} expose deployment nodejs-deployment --name nodejs-service
kubectl -n ${NS} get service
``` 

Publish externally your application using the free wildcard DNS nip.io for the URL: 
``` bash
kubectl -n ${NS} create ingress nodejs-ingress --class nginx --rule "nodejs-${USER}.${PUBLIC_IP}.nip.io/*=nodejs-service:8080" --dry-run=client -o yaml
kubectl -n ${NS} create ingress nodejs-ingress --class nginx --rule "nodejs-${USER}.${PUBLIC_IP}.nip.io/*=nodejs-service:8080"
kubectl -n ${NS} get ingress
```

Now you should be able to browse your brand new nodejs application: 
``` bash
echo "Open your browser an paste the following URL: http://nodejs-${USER}.${PUBLIC_IP}.nip.io"
```

