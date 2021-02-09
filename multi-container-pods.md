# Multi-container Pods (10%)
In this section I want to review how to connect multiple containers in a single pod. 

- Create a Pod with two containers, both with image busyboxy and command "echo hello; sleep 3600." Connect to the second tainer and run ls.
    - The commands I ran: 
        ```bash
        # First created a pod with a dry run
        kk run -it --image busybox --rm --restart=Never --dry-run=client -o yaml test
        ```
        Next I edited the yaml file to look like below. I didn't know how to create 
        a muticontainer from the command line so I took this approach. 
        ```yaml
        apiVersion: v1
            kind: Pod
            metadata:
            creationTimestamp: null
            labels:
                run: busybox
            name: busybox
            spec:
            containers:
            - command: ["/bin/sh"]
                args: ["-c", "echo hi; sleep 3600"]
                image: busybox
                name: busybox1
            - command: ["/bin/sh"]
                args: ["-c", "echo hi; sleep 3600"]
                image: busybox
                name: busybox2
                resources: {}
            dnsPolicy: ClusterFirst
            restartPolicy: Always
            status: {}
        ```
        Next I used the command line to create the pod: 
        ```bash
        kk apply -f multicont.yaml
        ```
        Next to exec into a specific pod, I used the following: 
        ```bash
        kk exec -it busybox -c busybox1 -- sh #exec into the container and run ls
        ```
    - Answers: 

        Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

        ```bash
        kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
        vi pod.yaml
        ```

        Copy/paste the container related values, so your final YAML should contain the following two containers (make sure those containers have a different name):

        ```YAML
        containers:
        - args:
            - /bin/sh
            - -c
            - echo hello;sleep 3600
            image: busybox
            imagePullPolicy: IfNotPresent
            name: busybox
            resources: {}
        - args:
            - /bin/sh
            - -c
            - echo hello;sleep 3600
            image: busybox
            name: busybox2
        ```

        ```bash
        kubectl create -f pod.yaml
        # Connect to the busybox2 container within the pod
        kubectl exec -it busybox -c busybox2 -- /bin/sh
        ls
        exit

        # or you can do the above with just an one-liner
        kubectl exec -it busybox -c busybox2 -- ls

        # you can do some cleanup
        kubectl delete po busybox
- Create pod with nginx container exposed at port 80. Add a busybox init container which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". Make a volume of type emtpyDir and mount it in both containres. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create  busybox and run "wget -O- IP". 
    - Commands I ran:

        1. Create a pod the same way as I did in the previous problem
        2. Edit the Yaml file in the following way: 
            ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
            creationTimestamp: null
            labels:
                run: nginx
            name: nginx
            spec:
            containers:
            - image: nginx
                name: nginx
                ports:
                - containerPort: 80
                resources: {}
                volumeMounts: 
                - mountPath: /usr/share/nginx/html
                name: resources
            initContainers: 
            - name: downloader
                image: busybox
                command: ["sh", "-c", "wget -O /resources/index.html http://neverssl.com/online"]
                volumeMounts: 
                - mountPath: /resources
                name: resources
            volumes: 
            - name: resources
                emptyDir: {}
            dnsPolicy: ClusterFirst
            restartPolicy: Always
            status: {}
            ```
        3. Finally, I can fetch the data from the nginx container with the following command: 
            ```bash
            kk run -it --restart=Never --image busybox --rm fetcher -- wget -O- 10.42.0.76 # After retrieving the IP address. 
            ```
    - Answers: 
        Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

        ```bash
        kubectl run web --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
        ```

        Copy/paste the container related values, so your final YAML should contain the volume and the initContainer:

        Volume:

        ```YAML
        containers:
        - image: nginx
        ...
            volumeMounts:
            - name: vol
            mountPath: /usr/share/nginx/html
        volumes:
        - name: vol
            emptyDir: {}
        ```

        initContainer:

        ```YAML
        ...
        initContainers:
        - args:
        - /bin/sh
        - -c
        - wget -O /work-dir/index.html http://neverssl.com/online
        image: busybox
        name: box
        volumeMounts:
        - name: vol
            mountPath: /work-dir
        ```

        In total you get:

        ```YAML

        apiVersion: v1
        kind: Pod
        metadata:
        labels:
            run: box
        name: box
        spec:
        initContainers: #
        - args: #
            - /bin/sh #
            - -c #
            - wget -O /work-dir/index.html http://neverssl.com/online #
            image: busybox #
            name: box #
            volumeMounts: #
            - name: vol #
            mountPath: /work-dir #
        containers:
        - image: nginx
            name: nginx
            ports:
            - containerPort: 80
            volumeMounts: #
            - name: vol #
            mountPath: /usr/share/nginx/html #
        volumes: #
        - name: vol #
            emptyDir: {} #
        ```

        ```bash
        # Apply pod
        kubectl apply -f pod-init.yaml

        # Get IP
        kubectl get po -o wide

        # Execute wget
        kubectl run box --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- IP"

        # you can do some cleanup
        kubectl delete po box
        ```
    - Discussion: The methods are basically the same except for a few minor changes. 