# Core concepts
In this section we will look at: 
- Kubernetes API and primitives
- Create and configure basic Pods

## Helpful links: 
- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Study Guide](https://github.com/dgkanatsios/CKAD-exercises/blob/master/a.core_concepts.md)
- [Kubernetes v1.19 API Documentation](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/)

## Exercises
- Create a namespace called 'mynamespace' and a pod with the image 'nginx' called nginx in this namespace: 
    - Commands I ran: 
        ```
        kk create namespace mynamespace; 
        kk  run --image=nginx:latest -n mynamespace nginx
        ```
    - Answers: 
        ```
        kubectl create namespace mynamespace
        kubectl run nginx --image=nginx --restart=Never -n mynamespace
        ```
    - **Discussion**: The main difference with the way I ran the pod and the way the pod *should* be run, is that I didn't include the restart policy. By default, the restart policy [is set to 'Always'](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy). So the pod would continue to restart itself if there was an error. 
- Create the pod that was just described using YAML
    - Commands I ran: 
        ```
        kk get po -n mynamespace nginx -o yaml >> pod.yaml
        kk delete po nginx -n mynamespace
        kk apply -f pod.yaml
        ```
    - Answers: 
        ```
        kubectl run nginx --image=nginx --restart=Never --dry-run=client -n mynamespace -o yaml > pod.yaml
        kubectl create -f pod.yaml -n mynamespace
        ```
    - **Discussion:** This question was a bit ambiguous, the purpose was really to create a yaml file for a pod like the previous question, but not run the pod until you apply the yaml file. The mistake I made here is that I assumed that the pod as was already running and could therefore get the yaml file. So in the future, I can call `run` and give it the `dry-run` flag as well as the `-o` flag to output a yaml file. 
- Create a busybox pod (using the kubectl command) that runs the command "env." Run it and see the output. 
    - Commands I ran: 
        ```
        kk run --image busybox:latest --restart never busybox --command -- env
        kk logs busyboxy
        ```
    - Answers: 
        ```
        kubectl run busybox --image=busybox --command --restart=Never -it -- env
        ```
    - **Discussion**: Again, my method is valid here but it can be done in fewer steps. The trick to this one is that I can actually run the container in *interactive* mode by using the flags `-it` which allow me to see the out of the container immediately. 

- Create a busybox pod (using YAML) that runs the command "env." Run it and see the output. 
    - Commands I ran: 
        ```
        kk run --image busybox:latest --restart Never --dry-run=client -o yaml busybox --command -- env > pod.yaml
        kk create -f pod.yaml
        kk logs busy boxy
        ```
    - Answers: 
        ```
        kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > envpod.yaml
        kubectl apply -f envpod.yaml
        kubectl logs busybox
        ```
- Get the yaml for a new namespace called 'myns' without creating it. 
    - Commands I ran: 
        ```
        kk create ns myns --dry-run=client -o yaml > ns.yaml
        ```
    - Answers: 
        ```kubectl 
        create namespace myns -o yaml --dry-run=client
        ```
- Get the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it. 
    - Commands I ran: 
        ``` bash
        kk create quota --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml myrq
        ```
    - Answers: 
        ```bash
        kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
        ```
- Get pods on all namespaces. 
    - Commands I ran: 
        ```bash
        kk get po --all-namespaces
        ```
    - Answers: 
        ```bash
        kubectl get po -A
        ```
- Create a pod with image nginx called gnxin and expose traffic on port 80
    - Commands I ran: 
        ```bash
        kk run --image=nginx --port 80 nginx
        ```
    - Answer: 
        ```bash
        kubectl run nginx --image=nginx --restart=Never --port=80
        ```
    - Disucssion: Again the restart never was used, but I don't think it will be necessary on the test unless they explicitly as for something like that. 
- Change pod's image to nginx:1.7.1. Observe that the container will b3e restarted as soon as the image gets pulled. 
    - Commands I ran: 
        ```bash
        kk edit po nginx
        ```
        - I then edited the image to be version 1.7.1. 
    - Answers: 
        ```bash
        # kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
        kubectl set image pod/nginx nginx=nginx:1.7.1
        kubectl describe po nginx # you will see an event 'Container will be killed and recreated'
        kubectl get po nginx -w # watch it
        ```
    - Discussion: In their example, they set the image right away instead of having to edit the yaml directly, which is the method I used.
- Get nginx pod's ip create in the previous step, use a temp busy image to wget its '/'
    - Commands I ran: 
        ```bash
        kk describe po nginx | grep -i IP # to get the IP
        kk run -it busybox --image busybox  --command -- wget 10.42.0.53/ # This downloads the index.html
        ```
    - Answers: 
        ```bash
        kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
        # create a temp busybox pod
        kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
        ```
    - Discussion: The big difference with what I did versus what the answers has is that I was struggling with getting the index.html in interactive mode when running the busybox. With the official answer, I can get the output directly to stdout. 

- Get pod's Yaml
    - Commands I ran: 
        ```bash
        kk get po nginx -o yaml
        ```
    - Answer: 
        ```bash
        kubectl get po nginx -o yaml
        # or
        kubectl get po nginx -oyaml
        # or
        kubectl get po nginx --output yaml
        # or
        kubectl get po nginx --output=yaml
        ```

- Get information about the pod, including details about potential issues (e.g. pod hasn't started)
    - Commands I ran: 
        ```bash
        kk describe po nginx 
        ```
    - Answers: 
        ```bash
        kubectl describe po nginx
        ```
- Get pod logs: 
    - Commands I ran: 
        ```bash
        kk logs -f nginx
        ```
    - Answers: 
        ```bash
        kubectl logs nginx
        ```
- If pod crashed and restarted, get logs about the previous instance
    - Commands I ran: 
        ```bash
        kk logs -p nginx
        ```
    - Answers: 
        ```bash
        kubectl logs nginx -p
        ```
- Execute a simple shell on the nginx pod: 
    - Commands I ran: 
        ```bash
        kk exec -it nginx -- bash
        ```
    - Answers: 
        ```bash
        kubectl exec -it nginx -- /bin/sh
        ```
    - Discussion: The advantage of /bin/sh is that the shell on the system is used instead of bash/csh/zsh. 
- Create a busybox pod that echos 'hello world' and then exits
    - Commands I ran: 
        ```bash
         kk run --restart=Never --rm -it --image busybox busybox -- echo 'hello world'
        ```
    - Answers:
        ```bash
        kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
        # or
        kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
        ``` 
- Do the same, but have the pod delete automatically when it's completed
    - Commands I ran: 
        ```bash
        kk run --restart=Never --rm -it --image busybox busybox -- echo 'hello world'
        ```
    - Answers: 
        ```bash
        kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
        ```
- Create an nginx pod and set an env value as `var=va1`. Check the env value existence within the pod.
    - Commands I ran: 
        ```bash
        kk run --env var1=var1 --restart=Never  --image busybox busybox
        kk describe po busybox # look for var1 in "Environment"
        ```
    - Answers: 
        ```bash
        kubectl run nginx --image=nginx --restart=Never --env=var1=val1
        # then
        kubectl exec -it nginx -- env
        # or
        kubectl exec -it nginx -- sh -c 'echo $var1'
        # or
        kubectl describe po nginx | grep val1
        # or
        kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
        ```
    


