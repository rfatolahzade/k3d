# k3d
Install k3d:
```bash
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
#OR
brew install k3d
```

Run a cluster:
Single node(plane):

```bash
k3d cluster create demo
```
With 3 nodes:
```bash
k3d cluster create --agents 2 demo  
```
Delete a cluster:
```bash
k3d cluster delete demo
```

Run advance cluster:
In my case I need port 80 and 443 and volume "/ParesDatabase"
```bash
k3d cluster create demohttps --agents 2  --port "443:443@loadbalancer"  --port "80:80@loadbalancer" --volume /ParesDatabase:/ParesDatabase
```
Other way to add a node with this command "k3d  node create NODENAME -c CLUSTERNAME"
```bash
k3d  node create newagent -c demo
```

Let's deploy our applictaion:
```bash
git clone https://github.com/rfinland/ParseChart.git
cd ParseChart/charts/parse
```
Change the replicas to 3:
```bash
nano /templates/server-deployment.yaml
```
As below:
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    parse.service: server
  name: server
spec:
  replicas: 3
  selector:
    matchLabels:
     parse.service: server
.
.
```

And then install the chart:
```bash
helm install parse . 
```
The Parse is up 
```bash
#HHTPS
curl https://YOUR-PARSE-URL/parse/health 
#HTTP
curl http://YOUR-PARSE-URL/parse/health 
```

List of nodes:
```bash
k3d node ls
kubectl get nodes --output wide
```
In this scenario I'll delete a node:
```bash
kubectl delete node k3d-k3s-default-agent-0
```
This cast shows you what happens after
https://asciinema.org/a/1zdK14TYdqgqB41VvBbVbeuj9


# HA
First, Kubernetes HA has two possible setups: embedded or external database (DB). We’ll use the embedded DB setup.
Second, K3s has two different technologies for HA with an embedded DB: one based on dqlite (K3s v1.18) and another on etcd (K3S v1.19+).
This means etcd is the default in the current K3s stable release and is the one which will be used in this blog. dqlite has been deprecated.
At the time of this writing, k3d was using K3s version v1.18.9-k3s1 by default. You can check this by running k3d version:

```bash
k3d version
```
OutPut:
```bash
k3d version v5.3.0
k3s version v1.22.6-k3s1 (default)
```
Let’s create our first K3s HA cluster with k3d,we should create an HA cluster with at least three control plane nodes.
We can achieve that with k3d in one command:
```bash
k3d cluster create --servers 3 
```
Checkout:
```bash
kubectl get nodes --output wide
kubectl get all --all-namespaces
kubectl get pods --all-namespaces --output wide
```
Now we have the base of our HA cluster. Let’s add an additional control plane node, bring some “destruction” and see how the cluster behaves.
Now we can quickly simulate the addition of another control plane node(--role=server) to the HA cluster:
```bash
k3d node create extraCPnode --role=server
```

With this additional node added, we can perform our final test: bring down the node0(k3d-k3s-default-server-0)!
HA: Heavy Armored against crashes
There are various reasons that this test is one of the most sensible. The node0 is normally the one that our KUBECONFIG refers to (in terms of IP or hostname), and therefore our kubectl application tries to connect to for running the different commands.
As we are working with containers, the best way to “crash” a node is to literally stop the container:
```bash
docker ps 
docker stop k3d-k3s-default-server-0
docker ps -a
```
And like magic, our cluster still responds to our commands using kubectl.
Now it is a good time to reference again the load balancer k3d uses and how it is critical in allowing us to continue accessing the K3s cluster.
While the load balancer internally switched to the next available node, from an external connectivity point of view, we still use the same IP/host. This abstraction saves us quite some efforts and it’s one of the most useful features of k3d.
Let’s look at the state of the cluster:
```bash
kubectl get all --all-namespaces
```
Everything looks right. If we look at the pods more specifically, then we will see that K3s automatically self-healed by recreating pods running on the failed node on other nodes:
Finally, to show the power of HA and how K3s manages it, let’s restart the node0 and see it being re-included into the cluster as if nothing happened:
```bash
docker start k3d-k3s-default-server-0
k3d node list
kubectl get pods --all-namespaces --output wide
```
Our cluster is stable, and all the nodes are fully operational again.

