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
Waiting for link "west-1" to become active...
Link "west-1" is active
You can now delete token.yaml.  It is now expired.

$ skupper link get
NAME      STATUS   COST
south-1   Error    10
west-1    Active   1

$ skupper link get west-1
NAME     STATUS   COST
west-1   Active   1

$ skupper link get west-1 -o yaml
apiVersion: v2alpha1
kind: Link
[...]

$ skupper link delete west-1
Waiting for link "west-1" to delete...
Link "west-1" is deleted
~~~

## Example listener operations

~~~ console
$ skupper listener create database:5432
Waiting for listener...
Listener "database-1" is ready

$ skupper listener get
NAME         HOST       PORT    ROUTING-KEY
payments-1   payments   8080    payments-8080
database-1   database   5432    database-5432

$ skupper listener get database-1 -o yaml
apiVersion: v2alpha1
kind: Listener
[...]

$ skupper listener set database-1 --port 5431
Waiting for listener...
Listener "database-1" is ready

$ skupper listener delete database-1
Listener "database-1" is deleted
~~~

## Example connector operations

~~~ console
$ skupper connector create database:5432 deployment/database
Waiting for connector...
Connector "database-1" is ready

$ skupper connector get
NAME         HOST       PORT    ROUTING-KEY     SELECTOR
payments-1   payments   8080    payments-8080   app=payments
database-1   database   5432    database-5432   app=database

$ skupper connector get database-1 -o yaml
apiVersion: v2alpha1
kind: Connector
[...]

$ skupper connector set database-1 --port 5431
Waiting for connector...
Connector "database-1" is ready

$ skupper connector delete database-1
Connector "database-1" is deleted
~~~

## Skupper resource commands

These are the core Skupper commands.  They are not the only commands,
however.  Additional commands will be added to the spec in the future.

### `skupper <resource-type>`

Print help text for the operations of this resource type.

### `skupper <resource-type> create [options]`

Create a resource.

Resource options are set using one or more `--some-key some-value`
command line options.  YAML resource options in camel case (`someKey`)
are exposed as hyphenated names (`some-key`) when used as command line
options.

`site create` blocks until the site is ready, including ingress if
configured.

`link create` blocks until the link is active.

`listener create` blocks until the router listener is ready to accept
connections.

`connector create` blocks until the router connector is ready to make
connections.

### `skupper <resource-type> delete <resource-name>`

Delete a resource.

Since site is a singleton, the resource name argument is not required
for site deletion.

The delete operation blocks until the resource is removed.

### `skupper <resource-type> get [<resource-name>]`

This works just like `kubectl get <type>/<name>`.  In the first
iteration of the CLI, I think we should just delegate to kubectl.

`get` without a qualifying resource name argument enumerates all the
resources of this type.  (Same as `kubectl get`.)

`get` with a resource name and `--output yaml` gives you the full
resource YAML.  (Same as `kubectl get`.)

### `skupper <resource-type> set <resource-name> [options]`

Set resource options.

`set` is very similar to `create`.  Instead of creating a new resource
from defaults, it updates an existing one.

This takes one or more long options (`--name foo`).  They are the same
options as those used on create.

Since site is a singleton, the resource name argument is not required
for setting site options.

`set` blocks with the same rules expressed for `create`.

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

## Thoughts

It's important that the Hello World steps work as scripted with no
sleeps or additional logic to wait on conditions.  That's why I have
the notes about how operations block.

By default, no ingress.  Overall, it seems a bit better to require
people specify when they want it.

Can you edit the name of a CR?

## Guidelines

* Commands block until the user's desired state is achieved.
* No blocking on user input.
