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
```
In this scenario I'll delete a node:
```bash
kubectl delete node k3d-demohttps-agent-1
```
This cast shows you what happens after
https://asciinema.org/a/1zdK14TYdqgqB41VvBbVbeuj9
