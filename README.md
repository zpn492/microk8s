# microk8s
A place to establish knowledge about microk8s and argocd

# Your microk8s solution with argo cd

 Required: snap
 
 **[install microk8s](#install-microk8s)** <br />
 **[deploy argocd](#deploy-argocd)** <br />
 **[install argocd cli](#install-arogcd-cli)** <br />
 **[deploy simple service](#deploy-simple-service)** <br />
 **[ssh access to github repository](#ssh-github)** <br />
 **[deploy keycloak](#deploy-keycloak)** <br />
 
## install microk8s

https://kubernetes.io/blog/2019/11/26/running-kubernetes-locally-on-linux-with-microk8s/

```
snap install microk8s --classic
```

### set an alias
```
snap alias microk8s.kubectl kubectl
```

### Enable dns and storage
```
microk8s enable dashboard dns storage
```

### token for dashboard
```
microk8s dashboard-proxy
```

## deploy argocd 
**[back](#microk8s)** <br />

https://argo-cd.readthedocs.io/en/stable/getting_started/

### create a namespace for argocd
```
kubectl create namespace argocd
```

### install argocd
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### to access argo cd ui, change argocd-server to loadbalancer

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer", "externalIPs":["192.168.1.27"]}}'
```


### get services
```
kubectl get svc -n argocd
```

### get password
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

## install arogcd cli
**[back](#microk8s)** <br />

https://argo-cd.readthedocs.io/en/stable/cli_installation/

### retrieve installation file
```
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```

### make it executeable and move it
```
chmod +x argocd
```
```
mv argocd /user/local/bin/
```

### login to argocd-server
```
argocd login <cluster-ip> --username admin --password <pass> --insecure
```

## deploy simple service 
**[back](#microk8s)** <br />

https://github.com/argoproj/argocd-example-apps/tree/master

```
argocd app create guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace default
```
### sync dummy app
```
argocd app sync guestbook
```
### delate dummy app
```
argocd app delete guestbook
```

## ssh github
**[back](#microk8s)** <br />

https://dev.to/hbolajraf/git-connecting-to-github-and-pushing-changes-using-ssh-on-windows-2f5

### generate ssh public/private keys
```
ssh-keygen -t rsa -b 4096 -C "<user>@gmail.com"
```

### place public key at your github account
> Copy public key
```
cat <key>.pub | clip
```
> Go to repository <br />
> Go to settings <br />
> Go to Security > Deploy keys
> Create a new deploy key

### add private key to your ssh agent
```
ssh-add <key>
```

### verify ssh connection
```
ssh -T git@github.com
```

## deploy keycloak 
**[back](#microk8s)** <br />

### create a new namespace for infrastructure
```
kubectl create namespace infrastructure
```

https://www.keycloak.org/getting-started/getting-started-kube

https://raw.githubusercontent.com/keycloak/keycloak-quickstarts/refs/heads/main/kubernetes/keycloak.yaml

```
argocd app create keycloak --repo https://github.com/zpn492/microk8s.git --path keycloak --dest-server https://kubernetes.default.svc --dest-namespace infrastructure
```
### sync keycloak app
```
argocd app sync keycloak
```

### lookup admin site
```
kubectl get svc -default
```

find cluster-ip and port-nr. from keycloak and copy paste it into your browser.

### find .well-known
<cluster-ip>:<port-nr>/realms/master/.well-known/openid-configuration

### Token retrieval after you created a client

https://datatracker.ietf.org/doc/html/rfc6749#section-4.4

```
curl -H "application/x-www-form-urlencoded" -d "grant_type=client_credentials&client_id=<client-id>&client_secret=HMqj.." http://<cluster-ip>:8080/realms/<realm>/protocol/openid-connect/token
```

### delate keycloak app
```
argocd app delete keycloak
```
