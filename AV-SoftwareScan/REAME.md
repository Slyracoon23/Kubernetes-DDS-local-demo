## INTRO
### In this scenario you'll learn how scan a machine using Clamav with Kubernetes #

 Deploying ClamAV in containers using kuberenetes Pods that uses DDS to publish its Data model. The resulting cluster allows kuberneretes features and   DDS features.

## STEP 1

### Build and Deploy the ClamAV microservice

To begin, make sure your Kubernetes environment is set up. Once the terminal has finished outputting messages and is ready for input it should be setup. To confirm it is ready please run the following command:

`kubectl version`{{execute}}

You should now see the versions of your kubectl client and server. If so, your environment is all set up. If you do not see the version of your Kubernetes server wait a few moments and repeat the previous command until it is shown.
 

Once the kubernetes is ready, you need to deploy ClavmAV to Kubernetes. To learn more about Kubernetes manifests, check out the following documentation: https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/

To do this use, the following command:

`kubectl apply -f clamav-deployment.yaml`{{execute}}

## STEP 2

### Viewing Your Deployment of ClamAV microservice

Great! You just deployed your first application by creating a deployment. This performed a few things for you:

- searched for a suitable node where an instance of the application could be run (we have only 1 available node)
- scheduled the application to run on that Node
- configured the cluster to reschedule the instance on a new Node when needed

To list your deployments use the `get deployments` command:

`kubectl get deployments`{{execute}}

We see that there is 1 deployment running a single instance of your app. The instance is running inside a Docker container on your node.

To view to logs of our pod, use the `logs <pod>` command:

`kubectl logs $(kubectl get pods --no-headers -o custom-columns=":metadata.name")`{{execute}}

The complicated `kubectl get pods --no-headers -o custom-columns=":metadata.name"` is getting your pod name.

As you can see, the application is scanning for any viruses on the host machine.

## STEP 3

### Viewing DDS Output

Pods that are running inside Kubernetes are running on a private, isolated network. By default they are visible from other pods and services within the same kubernetes cluster, but not outside that network. When we use kubectl, we're interacting through an API endpoint to communicate with our application.

To view to the output coming from our clamav pod, we can use the log command! Use the following command:

`kubectl logs $(kubectl get pods --no-headers -o custom-columns=":metadata.name")`{{execute}}

To view the DDS samples coming from our pod, we will create a temporary pod to see if our dds application is publishing topics! Use the following command:

`kubectl run dds-viewer --rm -i --tty --image earlpotters/dds -- bash`{{execute}}
(this might take a while)

Hopefully a shell popped up. Run the dds application spy to view the antiVirus output

`./bin/rtiddsspy -printSample`{{execute}}

You can now see the samples being published!

When you are done do Ctrl-C and run the following command to get back to the root shell:

`exit`{{execute}}

## STEP 4

### Updating Anti-Virus container

We have demonstrated how to run AV-software utilizing kubernetes and saw the DDS topic being published over the network but what if you want to make some changes? In this part we are going to update our ClamAV.

To view the current image version of the app, run a describe command against the Pods (look at the Image field): 

`kubectl describe pods`{{execute}}

To update the image of the application to  "new", use the set image command, followed by the deployment name and the new image version:

`kubectl set image deployments/clamav clamav=earlpotters/rticlamav:new`{{execute}}

The command notified the Deployment to use a different image for your app and initiated a rolling update. Check the status of the new Pods, and view the old one terminating with the get pods command:

`kubectl get pods`{{execute}}

## STEP 5

### Verify an update

First, let’s check that the App's description changed. To find out we can use describe deployment:

`kubectl describe deployments/clamav`{{execute}}


The update can be confirmed by running a rollout status command:

`kubectl rollout status deployments/clamav`{{execute}}

To view the current image version of the app, run a describe command against the Pods:

`kubectl describe pods`{{execute}}

We are now running a newer version of the app (look at the Image field)

## Step 6

### Rolling an update

Let’s perform another update, and deploy image tagged as 'broken' :

`kubectl set image deployments/clamav clamav=earlpotters/rticlamav:broken`{{execute}}

Use get deployments to see the status of the deployment:

`kubectl get deployments`{{execute}}

And something is wrong… We do not have running Pod available. List the Pods again:

`kubectl get pods`{{execute}}

A describe command on the Pods should give more insights:

`kubectl describe pods`{{execute}}

There is no image called broken in the repository. Let’s roll back to our previously working version. We’ll use the rollout undo command:

`kubectl rollout undo deployments/clamav`{{execute}}

The rollout command reverted the deployment to the previous known state (tag:'new' of the image). Updates are versioned and you can revert to any previously know state of a Deployment. List again the Pods:

`kubectl get pods`{{execute}}

Single Pod is running. Check again the image deployed on the them:

`kubectl describe pods`{{execute}}

We see that the deployment is using a stable version of the app ('new'). The Rollback was successful.



