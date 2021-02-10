# POD Design (20%)

Section covers how to design pods in kubernetes

## Create 3 pods with names nginx1, nginx2, and nginx3. All of them should have the label app=v1
- Commands I ran: 
    ```bash
    kk run --image nginx --labels app=v1 nginx1
    kk run --image nginx --labels app=v1 nginx2
    kk run --image nginx --labels app=v1 nginx3
    ```
- Answers: 
    ```bash
    kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
    kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
    kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
    ```
## Show all labels of the pods
- Commands I ran: 
    ```bash
    kk describe po | grep -i Labels
    ```
- Answers: 
    ```bash
    kubectl get po --show-labels
    ```
- Discussion: The show labels is much easier to read and you don't have to worry about grepping more lines if there are multiple labels. 

## Change the labels of pod 'nginx2' to be app=v2. 
- Commands I ran: 
    ```bash
    kk label pods nginx2 app=v2 --overwrite
    ```
- Answers: 
    ```bash
    kubectl label po nginx2 app=v2 --overwrite
    ```

## Get the label 'app' for the pods (show a column with APP labels)
- Commands I ran: 
    ```bash
    kk get po --show-labels | grep -i app
    ```
- Answers: 
    ```bash
    kubectl get po -L app
    # or
    kubectl get po --label-columns=app
    ```
- Disccusion: These commands are probably more reliable since you can associate pod with labels more reliably. 

## Get only the 'app=v2' pods
- Commands I ran: 
    ```bash
    # Also known as a selector
    kk get po -l app=v2
    ```
- Answers: 
    ```bash 
    kubectl get po -l app=v2
    # or
    kubectl get po -l 'app in (v2)'
    # or
    kubectl get po --selector=app=v2
    ```

## Remove the 'app' label from the pods we created before
- Commands I ran: 
    ```bash
    kk label po nginx1 app-
    kk label po nginx2 app-
    kk label po nginx3 app-
    ```
- Answers: 
    ```bash
    kubectl label po nginx1 nginx2 nginx3 app-
    # or
    kubectl label po nginx{1..3} app-
    # or
    kubectl label po -l app app-
    ```

## Create a pod that will be deployed to a node that has the label 'accelerator=nvidia-tesla-p100'
- Commands I ran: 
    ```bash
    # Create the label on a specific node
    kk label node some-node accelrator=nvidia-tesla-p100
    # Create a node (with a dry run)
    kk run --image nginx nginx-gpu --dry-run=client -oyaml > node-selector.yaml
    # Edit the node-selector.yaml file and add the 'nodeSelector' object to the spec
    ```
    ```yaml
    spec: 
        nodeSelector: 
            accelerator: nvidia-tesla-p100
    ```
- Answers: 

    Add the label to a node: 
    ```bash 
    kubectl label nodes <your-node-name> accelerator=nvidia-tesla-p100
    kubectl get nodes --show-labels
    ```

    We can use the 'nodeSelector' property on the Pod YAML: 
    ```bash
    apiVersion: v1
    kind: Pod
    metadata:
    name: cuda-test
    spec:
    containers:
        - name: cuda-test
        image: "k8s.gcr.io/cuda-vector-add:v0.1"
    nodeSelector: # add this
        accelerator: nvidia-tesla-p100 # the selection label
    ```
    You can easily find out where in the YAML it should be placed by: 
    ```bash
    kubectl explain po.spec
    ```

## Annotate pods nginx1, nginx2, nginx3 with "description='mydescription'" value
- Commands I ran: 
    ```bash
    kk annotate po nginx{1..3} description='mydescription'
    ```
- Answers: 
    ```bash 
    kubectl annotate po nginx{1..3} description='my description'
    ```
## Check the annotations for pod nginx1
- Commands I ran: 
    ```bash
     kk describe po nginx1 | grep -i annotations -A 5
    ```
- Answers: 
    ```bash
    kubectl describe po nginx1 | grep -i 'annotations'

    # or

    kubectl get pods -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
    ```
## Remove the annotations for these three pods
- Commands I ran: 
    ```bash
    kk annotate po nginx{1..3} --overwrite description-
    ```
- Answers: 
    ```bash
    kubectl annotate po nginx{1..3} description-
    ```
## Remove these pods to have a clean state in your cluster
- Commands I ran: 
    ```bash
     kk delete po nginx{1..3}
    ```
- Answers: 
    ```bash
    kubectl delete po nginx{1..3}
    ```
# Deployments

## Create a deployment with image nginx:1.7.8, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)
- Commands I ran: 
    ```bash 
    kk create deploy --image=nginx:1.7.8 nginx
    ```

    From here, I ran `kubeclt edit deploy nginx` and modified the replicas and the `ports` section of the yaml file. 

- Answers: 
    ```bash
    kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run=client -o yaml > deploy.yaml
    vi deploy.yaml
    # change the replicas field from 1 to 2
    # add this section to the container spec and save the deploy.yaml file
    # ports:
    #   - containerPort: 80
    kubectl apply -f deploy.yaml
    ```
    Another method: 
    ```bash
    kubectl create deploy nginx --image=nginx:1.7.8 --replicas=2 --port=80
    ```
- Discussion: For some reason, the last command here didn't work for me. I was getting the error "Error: unkown flag: --replicas". I think it may be a version issue. 

## View the YAML of this deployment
- Commands I ran: 
    ```bash
    kk get deploy nginx -o yaml
    ```
- Answers:
    ```bash
    kubectl get deploy nginx -o yaml
    ```

## View the YAML of the replica set that was created by this deployment
- Commands I ran: 
    ```bash
    kk get rs; 
    kk get rs nginx-asdfasd -o yaml
    ```
    We can see the ones that belong to the set with prefix "nginx" 

- Answers: 
    ```bash
    kubectl describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property
    # OR you can find rs directly by:
    kubectl get rs -l run=nginx # if you created deployment by 'run' command
    kubectl get rs -l app=nginx # if you created deployment by 'create' command
    # you could also just do kubectl get rs
    kubectl get rs nginx-7bf7478b77 -o yaml
    ```
## Get the YAML for one of the pods: 
- Commands I ran: 
    ```bash
    kk get po; 
    kk get po nginx-sksksksk -o yaml 
    ```
- Answers: 
    ```bash
    kubectl get po # get all the pods
    # OR you can find pods directly by:
    kubectl get po -l run=nginx # if you created deployment by 'run' command
    kubectl get po -l app=nginx # if you created deployment by 'create' command
    kubectl get po nginx-7bf7478b77-gjzp8 -o yaml
    ```

## Check how the deployment rollout is going
- Commands I ran: 
    ```bash
    kk get po -l app=nginx # Look at the status
    ```
- Answers: 
    ```bash
    kubectl rollout status deploy nginx
    ```
- Discussion: the nice thing about the answer above is that you can easily see what the state of the rollout is. In the commands I ran you have to infer the status a bit by looking at the values of the status of each pod. 

## Update the nginx image to nginx:1.7.9
- Commands I ran: 
    ```bash
    kk edit deploy nginx
    # Change the image to nginx:1.7.9
    ```
- Answers: 
    ```bash
    kubectl set image deploy nginx nginx=nginx:1.7.9
    # alternatively...
    kubectl edit deploy nginx # change the .spec.template.spec.containers[0].image
    ```
## Check the rollout history and confirm that the replicas are OK
- Commands I ran: 
    ```bash
    kk rollout history deploy nginx
    kk get rs -l app=nginx
    ```
- Answers: 
    ```bash
    kubectl rollout history deploy nginx
    kubectl get deploy nginx
    kubectl get rs # check that a new replica set has been created
    kubectl get po
    ```
- Discussion: The extra commands that are run the answers may be a result of just making sure that each step of a deployment is working correctly. 

## Undo the latest rollout and verify that new pods have the old image (nginx:1.7.8)
- Commands I ran: 
    ```bash
    kk rollout undo deploy/nginx
    ```
- Answers: 
    ```bash
    kubectl rollout undo deploy nginx
    # wait a bit
    kubectl get po # select one 'Running' Pod
    kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # should be nginx:1.7.8
    ```
## Do an on purpose update of the deployment with a wrong image: nginx:1.91
- Commands I ran: 
    ```bash
    kk edit deploy nginx
    # Edit the yaml to change the image to an incorrect one and save the file.
     ```
- Answers: 
    ```bash
    kubectl set image deploy nginx nginx=nginx:1.91
    # or
    kubectl edit deploy nginx
    # change the image to nginx:1.91
    # vim tip: type (without quotes) '/image' and Enter, to navigate quickly
    ```

## Verify that somethings wrong with the rollout
- Commands I ran 
    ```bash
    kk rollout status deploy nginx # This hangs 
    kk get deplohy nginx; # This shows that Update-To-Date is 1
    kk get po -l app=nginx # Shows the error "ImagePullBackOff"
    ```
- Answers: 
    ```bash
    kubectl rollout status deploy nginx
    # or
    kubectl get po # you'll see 'ErrImagePull'
    ```

## Return the deployment to the second revions (number 2) and verify the image is nginx:1.7.9
- Commands I ran: 
    ```bash
    kk rollout undo --to-revision=2 deploy nginx # This returned an error because revision 2 was not present. Not sure if this is an issue with how I undid my steps previously or if something is happening. 
    ```
- Answers: 
    ```bash 
    kubectl rollout undo deploy nginx --to-revision=2
    kubectl describe deploy nginx | grep Image:
    kubectl rollout status deploy nginx # Everything should be OK
    ```
- Discussion: Neither my commands nor the answers worked. I think the issue might have something to do with how I undid one of the revisions


## Check the details of the fifth revision (number 5)
- Commands I ran 
    ```bash
    kk rollout history deploy nginx --revision=5
    ```
- Answers: 
    ```bash
    kk rollout history deploy nginx --revision=5
    ```

## Scale the deployment to five replicas
- Commands I ran: 
    ```bash
    kk scale deploy nginx --replicas=5
    ```
- Answers: 
    ```bash
    kubectl scale deploy nginx --replicas=5
    kubectl get po
    kubectl describe deploy nginx
    ```

## Autoscale the deployment, pods between 5 and 10, targeting CPU utilization at 80%
- Commands I ran: 
    ```bash
    kk autoscale deployment nginx --cpu-percent=80 --min=5 --max=10
    ```
- Answers: 
    ```bash
    kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
    ```
- Discussion: I've never done this before so it is time to do some research. The technology used is something called a [Horizontal Pod Auotscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/). It is easy to set with a simple kubernetes command called `autoscale`. 

## Pause the rollout of the deployment
- Commands I ran: 
    ```bash
    kk rollout pause deploy nginx
    ```
- Answers: 
    ```bash
    kubectl rollout pause deploy nginx
    ```

## Update the image to nginx:1.9.1 and check that there's nothing going on, since we paused the rollout. 
- Commands I ran: 
    ```bash
    kk edit deploy/nginx 
    # Change the image name
    kk rollout status deploy/nginx #Verified that nothing is happenign
    kk get po # Noticed that no pods are terminating 
    kk describe po <podid> # Noticed that image is still set to the old one. 
    ```
- Answers:
    ```bash 
    kubectl set image deploy nginx nginx=nginx:1.9.1
    # or
    kubectl edit deploy nginx
    # change the image to nginx:1.9.1
    kubectl rollout history deploy nginx # no new revision
    ```

## Resume the rollout and check that the nginx:1.9.1 image has been applied
- Commands I ran: 
    ```bash
    kk rollout resume deploy/nginx
    kk rollout status deploy/nginx # Noticed that it was starting to rollout the new image
    kk get po # Saw the new pods 
    kk describe po nginx-678645bf77-jm4cd | grep -i image # Noticed the new image in the deployment for each pod.  
    ```
- Answers: 
    ```bash
    kubectl rollout resume deploy nginx
    kubectl rollout history deploy nginx
    kubectl rollout history deploy nginx --revision=6 # insert the number of your latest revision
    ```

## Delete the deployment and the horizontal pod autoscaler you created
- Commands I ran:
    ```bash
    kk delete deploy nginx
    ```
- Answers: 
    ```bash
    kubectl delete deploy nginx
    kubectl delete hpa nginx
    ```
- Discussion: In my commands, I did not delete the "horizontal autoscaler". The resource type for this thing is called "hpa" or "Horizontal Pod Autoscaler". 


# Jobs
## Create a job named pi with image perl that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"
- Commands I ran: 
    ```bash
    kk create job pi --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
    ```
- Answers: 
    ```bash
    kubectl create job pi  --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
    ```
## Wait until its done, then get the output
- Commands I ran: 
    ```bash
    kk get po # Figure out the name of the pod
    kk logs <pod id>
    ```
- Answers: 
    ```bash
    kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
    kubectl logs job/pi
    kubectl delete job pi
    ```

## Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'
- Commands I ran: 
    ```bash
    kk create job hello --image=busybox -- /bin/sh -c "echo hello;sleep 30;echo world"
    ```
- Answers: 
    ```bash
    kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
    ```
- Discussion: Be care about scripts that have ';' in them. The reason being is that the first time that I ran this command, I didn't use the `/bin/sh` command my local terminal did the sleeping. 

## Follow the logs for the pod (you'll wait for 30 seconds)
- Commands I ran: 
    ```bash
    kk logs -f <pod-id>
    ```
- Answers: 
    ```bash
    kubectl get po # find the job pod
    kubectl logs busybox-ptx58 -f # follow the logs
    ```

## See the status of the job, describe it and see the logs
- Commands I ran: 
    ```bash
    kk get job hello
    kk logs job/hello
    kk describe job hello
    ```
- Answers: 
    ```bash
    kubectl get jobs
    kubectl describe jobs busybox
    kubectl logs job/busybox
    ```

## Delete the job
- Commands I ran: 
    ```bash
    kk delete job hello
    ```
- Answers: 
    ```bash
    kubectl delete job busybox
    ```

## Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute. 
- Commands I ran: 
    ```bash
    kk create job hello --dry-run=client --image=busybox -oyaml  -- /bin/sh -c "echo hello;sleep 30;echo world" > deadline-job.yaml
    ```
    Then I edited the file and added:  
    ```yaml
    spec
         activeDeadlineSeconds: 30
    ```
    to the spec portion of the script

    ```bash
    kk apply -f deadline-job.yaml
    ```
- Answers: 
    ```bash
    kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
    vi job.yaml
    ```
    Add job.spec.activeDeadlineSeconds=30


## Create the same job, make it run 5 times, one after the other. Verify its status and delete it. 
- Commands I ran: do the same thing as above but set the number of completions to be 5. 
- Answers: Answer is the same. 

## Create the same job, make it run 5 times in parallel this time. 
- Commands I ran: Simply set parllelism to 5
- Answers: Yes

# Cron Jobs
## Create a cron job with image busybox that runs on a sechdule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output
- Commands I ran: 
    ```bash
    kk create cronjob --image=busybox --schedule="*/1 * * * *" busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
    ```
- Answers: 
    ```bash
    kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
    ```

## See its logs and delete it
- Commands I ran: 
    ```bash
    kk logs busybox-1612970820-x6mpl
    kk delete cronjob busybox
    ```
- Answers: 
    ```bash
    kubectl get cj
    kubectl get jobs --watch
    kubectl get po --show-labels # observe that the pods have a label that mentions their 'parent' job
    kubectl logs busybox-1529745840-m867r
    # Bear in mind that Kubernetes will run a new job/pod for each new cron job
    kubectl delete cj busybox
    ```

## Create a cron job with iamge busybox that runs every minitu and writes 'date; echo Hello from Kubernetes cluster' to standard output. the cron job should be terminiated if it takes more than 17 seconds to start after its schedule
- Commands I ran: 
    ```bash
    kk create cronjob --dry-run=client -oyaml --image=busybox --schedule="*/1 * * * *" busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster" > cronjob.yaml
    ```
    then edit the yaml file and add ` startingDeadlineSeconds: 16` to the spec. 

- Answers: The same as above. 

