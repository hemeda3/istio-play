# istio-play
istio-play-book


https://bani.com.br/2018/09/istio-sidecar-injection-enabling-automatic-injection-adding-exceptions-and-debugging/

Manual injection
Istio ships with a tool called istioctl. Yeah, it seems it’s inspired in some other loved tool :). One of its feature is the ability to inject the istio-proxy sidecar into your service Pod. Let’s use it, using a simple busybox pod as an example:

1- create file  busybox.yaml 

    command : vi  busybox.yaml  then :-
    apiVersion: v1
    kind: Pod
    metadata:
      name: busybox-test
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command: ['sh', '-c', 'echo Hello Kubernetes! && sleep infinity']`
 2- inject istio-envoy into our pods :
 ```shell
 istioctl kube-inject -f busybox.yaml > busybox-injected.yaml
      cat busybox-injected.yaml 
```
As you can see above, that command generated another yaml file, similar to the input (busybox pod) but with the sidecar (istio-proxy) added into the pod. So, in some way this is not a 100% manual job, right? It saves us a bunch of typing. You are ready to apply this modified yaml into your kubernetes cluster:


3-  apply/launch the 2 containers in 1 pod with this command:
```shell
kubectl apply -f busybox-injected.yaml
```
or, if you don't want to have an intermediate file, apply directly using the original file:
```shell
 kubectl apply -f <(istioctl kube-inject -f busybox.yaml)
```
4- check contaienr/pod log files,because this pod contains more than 1 container we can't just use kubectl logs PODNAME
becuase inside this POD we have two container 1- our applciation 2- side car containers contains our Envoy/Proxy dataplane
so we have to specify the container name after pod name like this :
```shell
kubectl logs busybox-test busybox-container 
```
```shell
kubectl logs busybox-test -c busybox-container

```

One natural question that might come is: Where does that data come from? How does it know that the sidecar image is docker.io/istio/proxyv2:1.0.2? The answer is simple: All that data comes from a ConfigMap that lives in the Istio control plane, on the istio-system namespace:

> You can edit this ConfigMap with the values you want to inject into your pods.

    As you can see, that is mostly a template that will get appended into your pod definition, f you want to use other image for the istio-proxy container, use other tag, or wish to tweak anything that will get injected, this is the stuff you need to edit. Remember that this ConfigMap is used for injection in all your pods in the service mesh. Be careful 🙂 Because istioctl reads a ConfigMap in order to know what to inject, that means you need to have access to a working Kubernetes cluster with Istio properly installed. If for some reason you don’t have such access, you can still use istioctl, by supplying a local configuration file:
```shell
# run this previosly, with proper access to the k8s cluster
  kubectl -n istio-system get configmap istio-sidecar-injector -o=jsonpath='{.data.config}' > inject-config.yaml
# feel free to modify that file, and you can run at any point later:
  istioctl kube-inject --injectConfigFile inject-config.yaml ...
```
  :::: Automatic injection::::
   The other way of having istio-proxy injected into your pods is by telling Istio to automatically do that for you. In fact, this is enabled by default for all namespaces with the label istio-injection=enabled. This means that, if a namespace has such label, all pods within it will get the istio-proxy sidecar automatically. You don’t need to run istioctl or do anything with your yaml files!
 The way it works is quite simple: It makes use of a Kubernetes feature called MutatingWebhook which consists in Kubernetes notifying Istio whenever a new pod is about to be created, and giving Istio the chance to modify the pod spec on the fly, just before actually creating that pod. Thus, Istio injects the istio-proxy sidecar using the template found in that ConfigMap we saw above.
