# microk8s
A place to establish knowledge about microk8s and argocd

# Your microk8s solution with argo cd

 Required: snap
 
 **[install microk8s](#install-microk8s)** <br />
 **[deploy argocd](#deploy-argocd)** <br />
 **[install argocd cli](#install-arogcd-cli)** <br />
 **[deploy simple service](#deploy-simple-service)** <br />
 **[ssh access to github repository](#ssh-github)** <br />
 
## install microk8s

https://kubernetes.io/blog/2019/11/26/running-kubernetes-locally-on-linux-with-microk8s/

```
sudo snap install microk8s --classic
```

### set an alias
```
sudo snap alias microk8s.kubectl kubectl
```

### Enable dns and storage
```
sudo microk8s enable dashboard dns storage
```

### token for dashboard
```
sudo microk8s dashboard-proxy
```

## deploy argocd 
**[back](#microk8s)**
https://argo-cd.readthedocs.io/en/stable/getting_started/

### create a namespace for argocd
```
sudo kubectl create namespace argocd
```

### install argocd
```
sudo kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### get services
```
sudo kubectl get svc -n argocd
```

### get password
```
sudo kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{data.password}" | base64 -d; echo
```

## install arogcd cli
**[back](#microk8s)**

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
sudo mv argocd /user/local/bin/
```

### login to argocd-server
```
argocd login <cluster-ip> --username admin --password <pass> --insecure
```

## deploy simple service 
**[back](#microk8s)**

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
**[back](#microk8s)**

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
