# istio-play
istio-play-book


https://bani.com.br/2018/09/istio-sidecar-injection-enabling-automatic-injection-adding-exceptions-and-debugging/

Manual injection
Istio ships with a tool called istioctl. Yeah, it seems it’s inspired in some other loved tool :). One of its feature is the ability to inject the istio-proxy sidecar into your service Pod. Let’s use it, using a simple busybox pod as an example:

1- create file  busybox.yaml 
---
# vi  busybox.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: busybox-test
spec:
  containers:
  - name: busybox-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep infinity']
 ---
 2- inject istio-envoy into our pods :
# istioctl kube-inject -f busybox.yaml > busybox-injected.yaml
# cat busybox-injected.yaml 
As you can see above, that command generated another yaml file, similar to the input (busybox pod) but with the sidecar (istio-proxy) added into the pod. So, in some way this is not a 100% manual job, right? It saves us a bunch of typing. You are ready to apply this modified yaml into your kubernetes cluster:

---
3-  apply/launch the 2 containers in 1 pod with this command:
$ kubectl apply -f busybox-injected.yaml
*** # or, if you don't want to have an intermediate file, apply directly using the original file:
$ kubectl apply -f <(istioctl kube-inject -f busybox.yaml)
---
4- check contaienr/pod log files,because this pod contains more than 1 container we can't just use kubectl logs PODNAME
becuase inside this POD we have two container 1- our applciation 2- side car containers contains our Envoy/Proxy dataplane
so we have to specify the container name after pod name like this :
$kubectl logs busybox-test busybox-container  or
$kubectl logs busybox-test -c busybox-container

