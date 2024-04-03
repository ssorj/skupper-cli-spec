# Skupper CLI spec

## Hello World, briefly

~~~ console
# Get the CLI

$ curl https://skupper.io/install.sh | curl

# West

$ export KUBECONFIG=~/.kube/config-west
$ kubectl apply -f https://skupper.io/install.yaml
$ kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend

$ skupper site create --ingress loadbalancer
$ skupper token create ~/token.yaml
$ skupper listener create backend:8080

# East

$ export KUBECONFIG=~/.kube/config-east
$ kubectl apply -f https://skupper.io/install.yaml
$ kubectl create deployment backend --image quay.io/skupper/hello-world-backend --replicas 3

$ skupper site create
$ skupper link create ~/token.yaml
$ skupper connector create backend:8080 deployment/backend
~~~

## Example site operations

~~~ console
$ skupper site
[Site-specific help text]

$ skupper site create --name west --ingress loadbalancer
Waiting for status...
Waiting for ingress...
Site "west" is ready

$ skupper site get
NAME   STATUS   INGRESS
west   Ready    loadbalancer

$ skupper site get -o yaml
apiVersion: v2alpha1
kind: Site
[...]

$ skupper site set --ingress route
Option "ingress" set to "route"
Waiting for ingress...
Site "west" is ready

$ skupper site delete
Waiting for site "west" to delete...
Site "west" is deleted
~~~

## Example token operations

~~~ console
$ skupper token
[Token-specific help text]

$ skupper token create token.yaml
Token file created at token.yaml
The token expires after 1 use or after 15 minutes
~~~

## Example link operations

~~~ console
$ skupper link
[Link-specific help text]

$ skupper link create token.yaml
Waiting for link "link-2" to become active...
Link "link-2" is active.  You can now delete token.yaml.  It is no longer usable.

$ skupper link get
NAME     STATUS   COST
link-1   Error    10
link-2   Active   1

$ skupper link get link-2
NAME     STATUS   COST
link-2   Active   1

$ skupper link get link-2 -o yaml
apiVersion: v2alpha1
kind: Link
[...]

$ skupper link delete link-2
Waiting for link "link-2" to delete...
Link "link-2" is deleted
~~~

## Example listener operations

~~~ console
$ skupper listener create database:5432
Waiting for listener "listener-2"...
Listener "listener-2" is ready

$ skupper listener get
NAME         HOST       PORT    ROUTING-KEY
listener-1   payments   8080    payments-8080
listener-2   database   5432    database-5432

$ skupper listener get listener-2 -o yaml
apiVersion: v2alpha1
kind: Listener
[...]

$ skupper listener set listener-2 --port 5431
Option "port" set to 5431

$ skupper listener delete listener-2
Listener "listener-2" deleted
~~~

## Thoughts

Consider doing platform install on demand in site create operation?

It's important that these work as scripted with no sleeps or condition waits.

Can you really edit the name of a CR?

By default, no ingress.  Overall, it seems a bit better to require
people specify when they want it.

## Rules

* No blocking on input.


## Skupper resource commands

### `skupper <resource-type>`

Get help about operations on this resource type.

### `skupper <resource-type> create`

Create a resource.

### `skupper <resource-type> delete <resource-name>`

Delete a resource.

Since site is a singleton, resource name is not required.

### `skupper <resource-type> get [<resource-name>]`

This works like kubectl get.  Get without a qualifying resource name
enumerates the resources of this type.

### `skupper <resource-type> set <resource-name> [settings]`

Set resource settings.

Since site is a singleton, resource name is not required.

### `skupper <resource-type> <special-operation>`

An operation specific to a particular resource type.

<!-- ## Skupper installation commands -->

<!-- ### `skupper install` -->

<!-- Kube: Equivalent to `kubectl apply -f https://skupper.io/install.yaml` -->

<!-- Blocks until: The Skupper controller is ready -->

<!-- Consider: Should this fall back to some static version of the install -->
<!-- YAML burned into the CLI?  For disconnected cases. -->

<!-- ### `skupper uninstall` -->

<!-- Kube: Equivalent to `kubectl delete -f https://skupper.io/install.yaml` -->

<!-- Blocks until: The Skupper resources are removed -->

<!-- ## skupper site -->

<!-- ## skupper site create -->

<!-- ## skupper site delete -->

<!-- ## skupper site get -->

<!-- ## skupper site set -->

<!-- ## skupper token -->

<!-- ### skupper token create <file> -->

<!-- ## skupper link -->

<!-- ### skupper link create <file> -->

<!-- ### skupper link delete <name> -->

<!-- ### skupper link get [<name>] -->

<!-- ## skupper listener -->

<!-- ### skupper listener create <host>:<port> -->

<!-- ### skupper listener delete -->

<!-- ### skupper listener get [<name>] -->

<!-- ### skupper listener set <name> [options] -->

<!-- ## skupper connector -->

<!-- ### skupper connector create -->

<!-- ### skupper connector delete -->

<!-- ### skupper connector get [<name>] -->

<!-- ### skupper connector set <name> [options] -->
