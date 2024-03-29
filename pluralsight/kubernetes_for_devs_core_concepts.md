# Kubernetes for Developers: Core Concepts

- Overview
  - Automates deployment, sacling, and management of containerized apps
  - Key Features
    - Service Discovery and Load Balancing
    - Storage orchestration
    - Automated, zero-downtime rollouts/rollbacks
    - Self-healing: Can bring containers back up when they die
    - Secrets and config management
    - Horizontal scaling
  - Nutshell: k8s syncs your desired state with the current state in the cluster
  - k8s manages a cluster of Nodes: Individual hosts, used to host one or more Pods
    - Master Node: holds `etcd` database; api server; controller manager; scheduler
      - Communicate via `kubectl`
      - Nodes have an agent that reports back to Master: `Kubelet`
      - `kube-proxy` to ensure network uniqueness, etc
      - Container runtime
    - Pods: Sets of containers
    - k8s Service: Resources to allow communication in/out of a Pod
  - Benefits / Use Cases
    - Containers
      - Local development - Entire local env, scripted out
      - Fewer app conflicts - Can run multiple versions easily
      - Env consistency: Better chance that if it works locally, it'll work in the cloud container runtime
    - k8s
      - Orchestration: Prod-ready Compose
      - Zero-downtime deploys
      - Self-healing: If a container goes down, can redeploy
      - Scaling can add Pods easily
    - Dev Use Cases for k8s
      - Better match Prod locally, if desired(e.g., minikube)
      - E2E test env is possible
      - Can easily test how your app works when scaled out / in
      - Secrets/ Config loading: Are they loading properly?
      - Perf testing against different scenarios
      - Multiple workloads: CI/CD, etc - If one crashes, bring another in
      - Different deployment options: Canary, blue/green, etc
      - Better able to help in Prod issues(e.g., assist DevOps engineers)
  - Running Locally
    - minikube: Easy, but has some limitations - Can't scale out in the same way as Prod, but have a Master and worker Node.
    - Docker desktop: Easiest to use since you'll have Docker anyway. Again, one Master and one worker limit.
      - Just check the box: "Enable Kubernetes"
    - kind: k8s in docker, allows scaling out
    - kubeadm: full k8s locally
  - kubectl
    - `version`
    - `cluster-info`
    - `get all` / `get _x_`
    - `run [name] --image=[name]` (simple deployment creation)
    - `port-forward [pod] [ports]` (to expose port access)
    - `expose` a port for a deployment / Pod
    - `create [resource]` / `apply [resource]`
  - Web UI dashboard
    - Can show Pod cpu/mem, service info, Workloads(pods, cron jobs, jobs, etc)
    - Can also pipe to e.g., Grafana
    - Not enabled by default, need to manually set it, then add users, etc.
- Pods
  - Basic execution unit
  - Pod IP, mem, volumes can be shared across multiple containers within pod
  - Scale horizontally
  - Are never "repaired" - If one dies, it is replaced completely
  - Share the same loopback(localhost)
  - Container processes need to bind to different ports within the pod
    - Ports can be reused in separate Pods
  - Pods never span nodes
  - Creation
    - `kubectl run`, but more usually `kubectl create/apply` via git-controlled yaml
    - By default, Pods are only accessible within the k8s cluster. For external access, must `kubectl port-forward`(e.g., `kubectl port-forward foo-pod 8080:80`).
      - Some commands(e.g., `port-forward`) will lock the console up
    - `kubectl delete pod foo-pod` to delete
      - but k8s will try to match current state! so will replace, in general.
      - must `kubectl delete deployment` to delete the Deployment and stop matching
  - Defining Pods with yaml
    - `apiVersion`: via k8s docs
    - `kind`: Resource type(e.g., `Pod`)
    - `metadata`: metadata about the Pod(e.g., `name`, `labels`)
    - `spec`: Blueprint(e.g., `containers` with their `name`s, `port`s, etc)
    - `kubectl create -f my.yml --dry-run --validate=true` (drop dry run/validate to do the real work)
      - If the Pod already exists, you'll get an error
      - `--save-config`: Store current properties in Resource's annotations for comparison in future `apply`s
      - `kubectl edit`: To edit via console
    - `kubectl delete` to remove
  - Pod health
    - Liveness Probe: Is it alive?
    - Readiness Probe: Is it ready to take requests?
    - Probe types
      - ExecAction: Execute an action, act on return code
      - TCPSocketAction: TCP check against container IP on a port
      - HttpGetAction: HTTP GET request
    - Results are: Success, Failure, Unknown
    - Failure
      - Can define a `FailureThreshold`
      - Default for a failing container would be to restart
- Deployments
  - Declarative way to manage Pods using a ReplicaSet
  - On Pod destroyed or Pod scale out, Deployments and ReplicaSets are used to true up the numbers
  - ReplicaSet
    - Self-healing mechanism
    - Ensure req number is met, including during scale-out
    - Provide fault tolerance
    - Replaces the need to create Pods manually
  - Deployment
    - Manages / wraps ReplicaSets
    - Scales ReplicaSets which scale Pods
    - Supports zero-downtime updates by creating/deleting Pods
    - Provides rollback
    - Issues a unique label to the ReplicaSet and Pods generated
    - yaml is very close to ReplicaSet
  - Creating a Deployment
    - yaml, `kind: Deployment`
    - `spec` with `selector`(Pod template labels) and `template`(used to create the Pod)
    - `kubectl create -f` / `kubectl apply -f`(but make sure to use `--save-config`)
    - `kubectl get deployment --show-labels` to list all deployments and their labels
    - `kubectl delete deployment [name]` to delete
    - `kubectl scale deployment [name] --replicas=n` to scale
  - Deployments in Action
    - `resources`: Defines constraints for e.g., cpu, memory
    - `kubectl get deployment`, `kubectl get deployments`, etc
  - Deployment Options and Zero Downtime
    - e.g., OS patching / upgrade
    - Rolling updates, blue-green, canary, are all available
    - Rolling: Existing instances => Fire up new Pod => Take down old one => Repeat until all new are in place
      - Thus, 0dt
    - `minReadySeconds`: Wait _n_ seconds to ensure that the app doesn't crash before allowing traffic
- Services
  - A single point of entry for accessing one or more Pods
  - Can's rely on Pod IPs since Pods are ephemeral, hence Service concept
    - Services are not ephemeral
    - Also Pod only gets an IP after it has been scheduled, so couldn't map it earlier
  - Service abstracts Pod IP from consumer, including Load Balancing, via endpoints  
  - Works at Network Layer 4
  - Service and Pods are tied together via labels
  - Much more to networking not covered here!
  - Service Types
    - ClusterIP: Default. Expose the service on a cluster-internal IP. For Pod<->Pod communication.
    - NodePort: Expose the service on each Node's IP at a static port. Port range is auto-allocated from 30000-32767 unless overidden. Each Node proxies the port(Client -> Node IP -> NodePort -> Service).
    - LoadBalancer: Provision an external IP to act as load balancer for service. Often combined with e.g., cloud provider's load balancer. Client -> LoadBalancer -> Node IP -> NodePort -> Service.
    - ExternalName: Maps a service to a DNS name. Service proxies requests to an external service, thus hiding any IP details for simpler changes. Service -> Pods -> ExternalName -> External service. When external changes, can just change the ExternalName service.
  - Creating
    - Can use `kubectl create -f ... --save-config` but obviously prefer yaml
    - `kubectl port-forward` to expose externally and route internally(e.g., for debugging)
    - `apiVersion`, `kind: Service`, `metadata`(e.g., name, labels, etc), `spec`(e.g., `type`, `selector`, `ports`)
    - Each Service name gets a DNS entry(e.g., `frontend` can access `backend` just using `backend:port`, igoring raw IPs entirely)
  - In Action
    - `kubectl get pods`, `kubectl get / describe pod [name]`(`-o yaml` if prefer yaml output)
    - With ClusterIP, can log into the pod and `curl ...my-service-clusterip:my-port` instead of relying on static IPs
    - With NodePort, can target `localhost:node-port` to access
    - With LoadBalancer, can target `localhost:load-balancer-port`
- Storage Options
  - Volumes for inter-Pod state / data sharing
  - Can survive Pod recycles if desired
  - A Pod can have multiple Volumes, attached via `mountPath`
  - Volumes
    - Types
      - emptyDir: Transient, if Pod goes away, so does this
      - hostPath: Pod mounts to Node's filesystem
        - If you want the Pod to pick up from where it left off after death, though, need Node affinity
      - nfs: Network File System share mounted into pod
      - configMap/secret: Provide Pod access to k8s resources
      - persistentVolumeClaim: Abstracted persistent storage
      - _tons_ of third party options that can be exposed(e.g., EBS, azureDisk, cephfs, etc)
    - Defined in Pod `spec` -> `volumes` with `name`, etc; then in `containers` via `volumeMounts`
    - `kubectl describe pod [name]` will expose volumes
  - PersistenVolumes and PersistentVolumeClaims
    - PersistentVolume: Cluster-wide storage unit, generally created by Cluster admin
      - Available to a Pod even if it gets rescheduled because it isn't tied to Pod lifetime
      - Relies on a storage provider(NFS, cloud storage, etc)
    - PersistentVolumeClaim: Requests usage of a PersistentVolume
    - yaml can be complicated - some sites will generate it for you
      - https://github.com/kubernetes/examples
      - `kind: PersistentVolumeClaim`, `accessModes` for read, write, ReadWriteOnce, etc.
  - StorageClasses
    - Storage template used to dynamically provision storage(i.e., Admin can set up SCs, then devs can request through a PVC, and auto-make the PV)
    - PVC references the SC, SC provisions the PV and binds it to the PVC
    - `kind: StorageClass`, give it a name, `reclaimPolicy` to retain or delete after PVC is dropped, `provisioner` if you want to auto-create(or `no-provisioner` if you want to create manually), `volumeBindingMode`(default is `immediate`, but can also wait til a claim is made)
    - `nodeAffinity` to set where PV will be created
    - PVC with `storageClassName` that references the template and asks for space, etc
    - In Pod, Deployment, etc can then define a `volume` and hook the PVC up to the one defined
    - `StatefulSet`: Manages deployment, scaling of a set of Pods, including guarantees around ordering, uniqueness, etc(e.g., for a database)
- ConfigMaps and Secrets
  - ConfigMaps: Inject config data into containers. Doesn't matter where Pods are mapped, etc.
    - File-based: Key as filename, value with content
    - via CLI
    - ConfigMap manifest .yml
    - Accessed via env vars _or_ via a ConfigMap volume
  - Creating
      - `kubectl create secrete generic [name] --from-literal=foo=bar` or `--from-file=ssh-privatekey=~/.....` or from a key pair `kubectl create secret tls tls-secret --cert=path/to/cert --key=/path/to/key`
      - What about defining with yaml? The file will only be base64 encoded, so accessible to anybody with access to the file. If needed, `kind: Secret`, `data` with keys and base64 encoded values(`foo: base64(bar)`)
    - Using
      - `kubectl get secrets`(or with `-o yaml`)
      - Accessible via env vars or file maps, just like ConfigMaps
      - `... valueFrom: secretKeyRef...` to map into Pod
- Putting it Together
  - See https://github.com/DanWahlin/CodeWithDanDockerServices
  - sample app: nginx as reverse proxy -> node.js app -> MongoDB + Redis
  - Both Redis and Mongo deployed into k8s(e.g., for Mongo, a StatefulSet / PVC is used for persistence)
  - Troubleshooting
    - `kubectl logs [pod-name]`, `kubectl logs [pod-name] -c [container-name]`, `kubectl logs -p [pod-name]`(for a Pod no longer running), `kubectl logs -f [pod-name]`(stream logs)
    - `kubectl describe pod [pod-name]`, `kubectl get pod [pod-name] -o yaml` or `kubectl get deployment [deployment-name] -o yaml`(to get all details including image, ip, etc)
    - `kubectl exec [pod-name] -it sh` to get a shell
    - e.g.,  `CreateContainerConfigError` -> `kubectl get pod...` -> Shows state "Password not found"(also in `describe`)
