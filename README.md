# Quick [kind](https://kind.sigs.k8s.io/) setup

_***disclaimer:** commands were tested on MacOS, so your setup might need slight compatibility adjustments_

## Install/setup kind

1. Get kind as described [here](https://kind.sigs.k8s.io/docs/user/quick-start#installation)
2. Create cluster with `kind create cluster --config kind-config.yaml`
    - [kind-config.yml](./kind-config.yaml) will create 1 control plane + 3 workers

Kind doesn't provide `start` and `stop` functionality for cluster which would be very useful...  
however here are handy commands for that on docker level.  
Just add alias to your `bashrc` or `zshrc` or `<whatever you use>`

```bash
alias kind_start='docker start $(docker container ls -aq --filter name=kind)'
alias kind_stop='docker stop $(docker container ls --filter name=kind -q)'
```

Additionally default restart policy on kind containers is `always`, we might want to change that...so run:

```bash
docker update --restart=unless-stopped $(docker container ls --filter name=kind -q)
```

## Install / Setup [Emissary](https://www.getambassador.io/docs/emissary/latest/tutorials/getting-started/)

1. Install [CRDS](./emissary/emissary-crds.yaml) with `kubectl apply -f emissary/emissary-crds.yaml`

2. Install [Emissary](./emissary/emissary-emissaryns.yaml) with `kubectl create namespace emissary && kubectl apply -f emissary/emissary-emissaryns.yaml`

    - _Default exposure for Emissary is `LoadBalancer` however we won't be using that for local setup, so it's commented out_
    - _There's a possbility to install [MetalLB](https://metallb.universe.tf/) beforehand as provider for LB type, however we won't be covering that here._

3. Install [Emissary Listener for port 80](./emissary/emissary-listener-80.yaml) with `kubectl apply -f emissary/emissary-listener-80.yaml`

_***note:** YAML definition files are unaltered to official guide version, except commenting out LB service part_

## Install [Demo app](https://github.com/hashicorp/http-echo)

1. Install [app](./banana-app.yaml) with `kubectl apply -f banana-app.yaml`
2. Create [basic service](banana-svc.yaml) for that app with `kubectl apply -f banana-svc.yaml`
3. Expose app with [NodePort Service](banana-np.yaml) with `kubectl apply -f banana-np.yaml`
4. Expose app with [Ingress Mapping](banana-mapping.yaml) with `kubectl apply -f banana-mapping.yaml`
5. For accessing app through exposed url you need to point your local DNS resolve to one of kind workers so:
    1. Get kind workers IP by executing:  

        ```bash
        for d in $(docker container ls -q --filter name="kind-worker"); \
        do \
            docker inspect $d --format '{{.NetworkSettings.Networks.kind.IPAddress}}'; \
        done
        ```

    2. Add `/etc/hosts` override for `banana.lk8s` pointing to one of those IP addresses
    3. App is available on `<node_ip>:30020` or `banana.lk8s`

## Important Note

_Due to way Docker works on Mac / Windows it's impossible to access Docker container internal IP without "hacky approach"._  
_On Mac tested and working is [this solution](https://github.com/chipmk/docker-mac-net-connect?ref=golangexample.com)._

_Additionally, if you'd like to avoid modifying `/etc/hosts` you can take a look how [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) works._
