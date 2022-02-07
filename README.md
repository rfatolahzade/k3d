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