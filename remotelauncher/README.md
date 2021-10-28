# In this scenario you'll learn how to use Remotelauncher to run commands and differenet machines 

## STEP 1
### Build and Deploy the Multi-Docker Environment


To begin, make sure your environment is set up. Run ls and see the following files are shown:
`docker-compose.yml`
`remotelauncher-demo`

`ls`{{execute}}

You should now see the the files. If so, your environment is all set up! Now run the following command to deploy your multi-docker environment!


`docker-compose up -d `{{execute}}

## STEP 2
### Start Remotelauncher

Hopefully you multi-docker enviroment is deployed, verify by running docker-compose ps:

`docker-compose ps`{{execute}}

You should see a container is up and running.

That container you just deployed is running a Remotelauncher Daemon that is able to do RPC calls.

To make your current host able to communicate with other machines you need to run the Remotelauncher application as well.

First, change direcotiry to the remotelauncher application.
`cd remotelauncher-demo/opt/rtiremotelauncher/`{{execute}}

Then, launch it in the background using the following command:

`./bin/rtiremotelauncherd -config ./etc/rtiremotelauncher/rtiremotelauncherd.conf &`{{execute}}

Verifiy it is running probably by running:

`ps`{{execute}}

## STEP 3
### Simple capabilities layout using Remote Launcher

Great, remotelauncher is running. Now we can go over all the things you can do with remotelauncher

Change directory to the Remotelauncher helper script:

`cd /root/remotelauncher-demo/rtirlc`{{execute}}

View all the machines that are running remotelauncher Daemon by running...

`node rtirlc.js ls -l`{{execute}}

You should see your host named host01 and a contianer hash. 

The ls commands show you allow the connected machines that remotelauncher is able to connect and run. In this case your docker container can run ping, spy and clamav.

Verify that the container hash is the same as docker ps:

`docker ps`{{execute}}

To list the full range of commands you can also do -h:

`node rtirlc.js -h`{{execute}}
## STEP 4 
### Example Clamav setup using Remote Launcher

Say you you have a machines that is running remotelauncher on your DDS setup. 
You see that your favorite AV-software, ClamAV, is available on that remote machine. 


Rather than accessing each machine individually you can now run commands over the DDS DataBus.

First change your directory to the rtilc helper script
`cd /root/remotelauncher-demo/rtirlc`{{execute}}

Now check which commands you can execuete:

`node rtirlc.js ls -l`{{execute}}

Looks like you can run Clamav!

First, export your docker_hash to an enviroment Varialbe:

`export DOCKER_HASH=$(docker ps -q)`{{execute}}

Let's start an instance:

`node rtirlc.js new --cmdline '--connextdds=ClamScanParticipantLibrary::ClamScanParticipant::ClamScanPublisher::AntiVirusWriter  --verbose --stdout  --log=/logs/clamscan.log  --recursive --database=/data .' $DOCKER_HASH/clamav`{{execute}}

Cool you kust created your instance. Remember, when an instance is created it starts in a stop state. Let's try to change the state to a RUN state.

`node rtirlc.js run -o  $DOCKER_HASH/clamav/1`{{execute}}

Your program will execute the command to completion. Do Ctrl-C to get out.
Let's see if the program is running by running ls again:

`node rtirlc.js ls -l`{{execute}}

You can see that it's in a running state. Well done!

To view the stdout, let's attach to the process


`node rtirlc.js attach -o  $DOCKER_HASH/clamav/1`{{execute}}


Cool right! Remotelaucnher has executed a process over DDS.

Checkout to see the DDS publications of remotelauncher using rtiddsspy:

`/root/remotelauncher-demo/bin/rtiddsspy -domainId 5`{{execute}}

## STEP 5
### Wrapup Remote Launcher

To cleanup remotelauncher properly, delete any processes on any remotemahcines you want to destroy using the command:

` node rtirlc.js delete $DOCKER_HASH/clamav/1`{{execute}}

Now, you can delete the machine following this command:

`docker-compose down`{{execute}}

Congratuations you were able to do a AV-Scan with remotelauncher!



