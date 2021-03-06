# k3sup-multipass

[k3sup](https://github.com/alexellis/k3sup) helper to supercharge [multipass](https://multipass.run/) based k3s instances, inspired by [this gist](https://gist.github.com/alexellis/85175164331f340d9860675f6af740f8)

## features
* creates multipass k3s instances with one command *and* transfers the kubeconfig to the host
* optionally proxies ingress controller to hosts localhost:80 (multipass does not have stable ip address support)

## demo

```
$ k3sup-multipass create test
Launched: k3s-test
x86_64
Downloading package https://github.com/alexellis/k3sup/releases/download/0.7.0/k3sup as /home/ubuntu/k3sup
Download complete.

<clipped k3sup output>

k3sup-multipass create done!

export KUBECONFIG=/Users/mpa/.kube/k3s-multipass-test
```

Let's set the *automagically* fetched kubeconfig
```
$ export KUBECONFIG=$(k3sup-multipass kubeconfig test)
```

And it works!
```
$ kubectl get node -o wide
kubectl get node -o wide
NAME       STATUS   ROLES    AGE   VERSION         INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k3s-test   Ready    master   44s   v1.16.3-k3s.2   192.168.64.14   <none>        Ubuntu 18.04.3 LTS   4.15.0-72-generic   containerd://1.3.0-k3s.5
```

Now, the instance IP changes every time (and multipass does not support fixed IP addresses...), so you need to find out the IP every time...
```
$ curl 192.168.64.14
404 page not found
```

Let's fix this with a small socat based proxy running at the good old 127.0.0.1
```
$ k3sup-multipass proxy:enable test
proxy to k3s-test enabled with --publish 127.0.0.1:80:80 --publish 127.0.0.1:443:443
```

And now our ingress works in the localhost (and you can use domains like test.localtest.me in your ingress objects)

```
$ curl localhost
404 page not found
```

We can free the port 80 temporarily with:
```
$ k3sup-multipass proxy:disable test
proxy disabled
```

And as expected, it's no longer accessible in localhost:
```
$ curl localhost
curl: (7) Failed to connect to localhost port 80: Connection refused
```

## install

```
git clone https://github.com/matti/k3sup-multipass
cd k3sup-multipass
ln -s $(pwd)/bin/k3sup-multipass /usr/local/bin
```

## usage

```
USAGE:
list
  lists all multipass k3s nodes

create NAME [--cpus=$(nproc) --disk=5G]
  creates a multipass k3s node

delete NAME
  deletes a multipass k3s node

kubeconfig NAME
  returns the path to the kubeconfig. use with: export KUBECONFIG=$(k3sup-multipass kubeconfig NAME)

proxy:enable NAME [--publish=127.0.0.1:80:80 --publish=127.0.0.1:443:443]
  publishes ports from the node to the host

proxy:disable NAME
  disables the proxy

proxy:show NAME
  shows the current proxy if enabled

ip NAME
  returns the IP address

shell NAME
  opens shell to the instance

exec NAME [-- command arg1 arg2]
  execs a command in the instance

kubectl|k NAME <kubectl args>
  runs kubectl with correct kubeconfig

helm NAME <helm args>
  runs helm with correct kubeconfig

version|--version
usage|help
```

# related
- https://github.com/matti/k3dolan
- https://github.com/matti/kindol

