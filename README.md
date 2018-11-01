# Introduction
This post demostrates the sidecar container.  A sidecar container is generally used to monitor something on the primary application and perform some task on that information alleviating the primary of that responsibility.  I have a little fun with the examples here.  The primary application writes a random number to a file which the sidecar container reads and displays one of two messages based on the content of the file.

## The Primary Container
This is just a simple container that writes a random number to a file.  The file is in a mounted volume that will be shared between containers.  

```console
  - name: app
    image: busybox
    imagePullPolicy: IfNotPresent
    command: [ "/bin/sh" ]
    args:
    - "-c"
    - "while true; do RAND=$(shuf -i 1-10 -n 1); echo ${RAND} > /var/log/num.log; sleep 5; done"
    volumeMounts:
    - name: vol-logs
      mountPath: /var/log
```

## The Sidecar Container
This container checks every five seconds to a file where the app container is writing a random number.  If the number is greater than '7' it displays good job.  The container could collect all the numbers and send them off, or do any number of tasks.  The point is to take care of the information, so that it doesn't have to be handled by the primary app.

```console
  - name: num-checker
    image: busybox
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: vol-logs
      mountPath: /var/log
    command:
    - "/bin/sh"
    - "-c"
    - "while true; do NUM=$(cat /var/log/num.log); if [ ${NUM} -gt 7 ]; then echo 'good job!'; else echo 'better luck next time'; fi; sleep 5; done"

```

## Execute the Pod
Here is the entire pod with both containers.  Each pod shares a volumeMount.  After creating the pod review the logs of the sidecar container.

```console
kind: Pod
apiVersion: v1
metadata:
  name: my-pod
  namespace: default
spec:
  volumes:
  - name: vol-logs
    emptyDir: {}
  containers:
  - name: app
    image: busybox
    imagePullPolicy: IfNotPresent
    command: [ "/bin/sh" ]
    args:
    - "-c"
    - "while true; do RAND=$(shuf -i 1-10 -n 1); echo ${RAND} > /var/log/num.log; sleep 5; done"
    volumeMounts:
    - name: vol-logs
      mountPath: /var/log
  - name: num-checker
    image: busybox
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: vol-logs
      mountPath: /var/log
    command:
    - "/bin/sh"
    - "-c"
    - "while true; do NUM=$(cat /var/log/num.log); if [ ${NUM} -gt 7 ]; then echo 'good job!'; else echo 'better luck next time'; fi; sleep 5; done"

```

Create the pod
```console
kubectl create -f my-pod.yml
pod/my-pod created
```

check the logs
```console
kubectl logs my-pod -c num-checker
better luck next time
better luck next time
better luck next time
better luck next time
good job!
better luck next time
```
