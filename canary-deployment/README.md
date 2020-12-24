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
kubectl apply -f ./nginx-deployment.yaml

## To view a list of deployments, execute the following command:
kubectl get deployments

## To scale the Pod down to one replica, execute the following command:
kubectl scale --replicas=1 deployment nginx-deployment

## To scale the Pod back up to three replicas, execute the following command:
kubectl scale --replicas=3 deployment nginx-deployment

## Trigger a deployment rollout and a deployment rollback
## A deployment's rollout is triggered if and only if the deployment's Pod template (that is, .spec.template ) is changed, 
## for example, if the labels or container images of the template are updated. Other updates, such as scaling the deployment,
## do not trigger a rollout.

## Trigger a deployment rollout
## To update the version of nginx in the deployment, execute the following command:
$ kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record

## The above updates the container image in your Deployment to nginx v1.9.1.

## To view the rollout status, execute the following command:
$ kubectl rollout status deployment.v1.apps/nginx-deployment

## View the rollout history of the deployment.
$ kubectl rollout history deployment nginx-deployment

## This is how the output will look:
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true

## Trigger a deployment rollback
## To roll back an object's rollout, you can use the kubectl rollout undo command.
## To roll back to the previous version of the nginx deployment, execute the following command:
$ kubectl rollout undo deployments nginx-deployment

## View the updated rollout history of the deployment.
$ kubectl rollout history deployment nginx-deployment

## This is how the output will look:
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
2         kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true
3         <none>

## View the details of the latest deployment revision
$ kubectl rollout history deployment/nginx-deployment --revision=3

## The output should look like the example. Your output might not be an exact match but it will show that the current
## revision has rolled back to nginx:1.7.9 .

## This is how the output will look:
deployment.apps/nginx-deployment with revision ##3
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

## Perform a canary deployment
## A canary deployment is a separate deployment used to test a new version of your application. A single service targets
## both the canary and the normal deployments. And it can direct a subset of users to the canary version to mitigate the
## risk of new releases. The manifest file nginx-canary.yaml that is provided for you deploys a single pod running a
## newer version of nginx than your main deployment. In this task, you create a canary deployment using this new deployment file.

## The manifest for the nginx Service you deployed in the previous task uses a label selector to target the Pods with
## the app: nginx label. Both the normal deployment and this new canary deployment have the app: nginx label. Inbound
## connections will be distributed by the service to both the normal and canary deployment Pods. The canary deployment
## has fewer replicas (Pods) than the normal deployment, and thus it is available to fewer users than the normal deployment.

## Create the canary deployment based on the configuration file by running:
$ kubectl apply -f nginx-canary.yaml

## When the deployment is complete, verify that both the nginx and the nginx-canary deployments are present. 
$ kubectl get deployments

## Switch back to the browser tab that is connected to the external LoadBalancer service ip and refresh the page. You
## should continue to see the standard "Welcome to nginx" page.
## Switch back to the Cloud Shell and scale down the primary deployment to 0 replicas.
$ kubectl scale --replicas=0 deployment nginx-deployment

## Verify that the only running replica is now the Canary deployment:
$ kubectl get deployments

## Switch back to the browser tab that is connected to the external LoadBalancer service ip and refresh the page. You
## should continue to see the standard "Welcome to nginx" page showing that the Service is automatically balancing
## traffic to the canary deployment.

## Note: Session affinity

## The Service configuration used in the lab does not ensure that all requests from a single client will always connect to
## the same Pod. Each request is treated separately and can connect to either the normal nginx deployment or to the
## nginx-canary deployment. This potential to switch between different versions may cause problems if there are
## significant changes in functionality in the canary release. To prevent this you can set the sessionAffinity field
## to ClientIP in the specification of the service if you need a client's first request to determine which Pod will be used for
## all subsequent connections.

