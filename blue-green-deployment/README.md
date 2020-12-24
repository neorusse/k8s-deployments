## Create a GKE Cluster 
$ gcloud container clusters create k8s-canary-deploy \
    --zone europe-west2-a

## Create a GKE Cluster 
$ gcloud container clusters create k8s-canary-deploy \
    --zone europe-west2-a

## Connect to the Cluster - Grant kubectl access to your cluster
$ gcloud container clusters get-credentials k8s-canary-deploy --zone europe-west2-a --project cloudkite-297703

## To view your Cluster credentials, run:
$ cat $HOME/.kube/config 

## Connect to the Cluster - Grant kubectl access to your cluster
$ gcloud container clusters get-credentials k8s-canary-deploy --zone europe-west2-a --project cloudkite-297703

## To view your Cluster credentials, run:
$ cat $HOME/.kube/config

## To deploy your manifest, execute the following command:
kubectl apply -f ./nginx-blue.yaml

## To view a list of deployments, execute the following command:
kubectl get deployments

## To scale the Pod down to one replica, execute the following command:
kubectl scale --replicas=1 deployment nginx-blue

## To scale the Pod back up to three replicas, execute the following command:
kubectl scale --replicas=3 deployment nginx-blue

## Trigger a deployment rollout and a deployment rollback
## A deployment's rollout is triggered if and only if the deployment's Pod template (that is, .spec.template ) is changed, 
## for example, if the labels or container images of the template are updated. Other updates, such as scaling the deployment,
## do not trigger a rollout.

## Trigger a deployment rollout
## To update the version of nginx in the deployment, execute the following command:
$ kubectl set image deployment.v1.apps/nginx-blue nginx=nginx:1.9.1 --record

## The above updates the container image in your Deployment to nginx v1.9.1.

## To view the rollout status, execute the following command:
$ kubectl rollout status deployment.v1.apps/nginx-blue

## View the rollout history of the deployment.
$ kubectl rollout history deployment nginx-blue

## This is how the output will look:
deployment.apps/nginx-blue 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment.v1.apps/nginx-blue nginx=nginx:1.9.1 --record=true

## Trigger a deployment rollback
## To roll back an object's rollout, you can use the kubectl rollout undo command.
## To roll back to the previous version of the nginx deployment, execute the following command:
$ kubectl rollout undo deployments nginx-blue

## View the updated rollout history of the deployment.
$ kubectl rollout history deployment nginx-blue

## This is how the output will look:
deployment.apps/nginx-blue
REVISION  CHANGE-CAUSE
2         kubectl set image deployment.v1.apps/nginx-blue nginx=nginx:1.9.1 --record=true
3         <none>

## View the details of the latest deployment revision
$ kubectl rollout history deployment/nginx-blue --revision=3

## The output should look like the example. Your output might not be an exact match but it will show that the current
## revision has rolled back to nginx:1.7.9 .

## This is how the output will look:
deployment.apps/nginx-blue with revision ##3
Pod Template:
  Labels:       app=nginx
        pod-template-hash=54f57cf6bf
  Containers:
   nginx:
    Image:      nginx:1.7.9
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>

## To deploy your manifest, execute the following command:
$ kubectl apply -f ./service-nginx.yaml

## To view the details of the nginx service, execute the following command:
$ kubectl get service nginx

## Perform a Blue-Green Deployment

## Create the green deployment based on the configuration file by running:
$ kubectl apply -f nginx-green.yaml

## When the deployment is complete, verify that both the nginx-blue and nginx-green deployments are present. 
$ kubectl get deployments

## Switch traffic from nginx-blue to nginx-green by updating the Nginx Service spec.selector.version 
## from v1 which targets nginx-blue deployment to v2 which targets nginx-green deployment
$ kubectl patch service nginx -p '{"spec":{"selector":{"version":"v2"}}}'

## Switch back to the browser tab that is connected to the external LoadBalancer service ip and refresh the page. You
## should continue to see the standard "Welcome to nginx" page.
## Switch back to the Cloud Shell and scale down the nginx-blue deployment to 0 replicas.
$ kubectl scale --replicas=0 deployment nginx-blue

## Verify that the only running replica is now the Nginx Green deployment:
$ kubectl get deployments

## Switch back to the browser tab that is connected to the external LoadBalancer service ip and refresh the page. You
## should continue to see the standard "Welcome to nginx" page showing that the Service is automatically balancing
## traffic to the canary deployment.

