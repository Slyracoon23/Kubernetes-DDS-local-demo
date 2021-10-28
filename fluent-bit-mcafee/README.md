# In this scenario you'll learn how apply DDS data modeling to native applications using kubernetes and Fluent-bit 

## STEP 1
### Build and Deploy the Mcafee microservice

To begin, make sure your Kubernetes environment is set up. Once the terminal has finished outputting messages and is ready for input it should be setup. To confirm it is ready please run the following command:

`kubectl version`{{execute}}

You should now see the versions of your kubectl client and server. If so, your environment is all set up. If you do not see the version of your Kubernetes server wait a few moments and repeat the previous command until it is shown.
 

Once the kubernetes is ready, you need to deploy your AV-software to Kubernetes. To learn more about Kubernetes manifests, check out the following documentation: https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/

In this example we will use Mcafee running in Docker.

To do this use, the following command:

`kubectl apply -f mcaffee-output.yaml`{{execute}}

##STEP 2

### Viewing Your Deployment of Mcaffe microservice

Great! You just deployed your AV application by creating a pod. The Mcafee container is now outputing scanning data to stdout. 

To take a look at what is outputing use the `logs <pod>` command:

`kubectl logs $(kubectl get pods --no-headers -o custom-columns=":metadata.name")`{{execute}}

We see that there a lot of data being published! Let's see if we can see the dds topics. Run the command:

`kubectl run dds-viewer --rm -i --tty --image earlpotters/dds -- bash`{{execute}}
(this might take a while)

Hopefully a shell poped up. Run the dds application spy to view the antiVirus output

`./bin/rtiddsspy -printSample`{{execute}}

...don't see anything? 

How can we publish over DDS without changing the application?

Please exit the shell and go back to your host shell

## STEP 3
### Transforming Log to DDS using Fluent-Bit

Fluent-bit is an open source data collector that lets you unify the data collection and consumption for a better use and understanding of data. In this stack Fluent Bit runs on each node (DaemonSet) and collects the logs from /var/logs and publishes them over DDS.

Note: In this demo, only the mcaffee log file was configured.

Let's start up Fluent-bit!

## STEP 4
### Running Fluent-Bit

First, let's create a new namespace for our DaemonSet Logger:

`kubectl create namespace logging`{{execute}}

Now we will give Fluent-Bit all permisions to the Kubernetes cluster

`kubectl apply -f fluent-bit-role-1.22.yaml -f fluent-bit-role-binding-1.22.yaml -f fluent-bit-service-account.yaml`{{execute}}

Once the permission are set, we can run fluent-bit's configmap and Daemonset

`kubectl apply -f fluent-bit-configmap.yaml -f fluent-bit-ds.yaml`{{execute}}

## STEP 5

### Viewing DDS

Let's take a look at our pods.

`kubectl get pods -A`{{execute}}

We can see that our fluent-bit daemon has launched. Let's see the logging details

`kubectl logs fluent-bit-<hash> -n logging`

Well done! We are parsing the output of the Mcafee! Let's see if DDS is being published.

`kubectl run dds-viewer --rm -i --tty --image earlpotters/dds -- bash`{{execute}}

`./bin/rtiddsspy -printSample`{{execute}}

## STEP 6
### Viewing DDS in Spunk

** TODO **


