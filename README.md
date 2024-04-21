# Training exercise 1 - Application with a database

In this exercise you will be tasked with creating a yaml descriptions such that
you will have an application and associated database running in your own
cluster.

You can either use https://gitlab.au.dk/swwao/demo-projs/typescript-mongodb-crud
as a starting point or, if you like, one of your own solutions from a previous
exercise. Do note that the everthing in the following will assume the above
mentioned application as the baseline. If you use something else, you have to
adjust!

The exercise is strictly the same as the previous except that we now have 2
applications in our cluster and these two need to communicate. Furthermore we
will employ _ConfigMaps_ and _Secrets_ where appropriate.  Environment variables
will thus be used to setup the DB and inform the _TypeScript_ app how to connect
to it.

Everything obviously must be in its own namespace as with the previous exercise.

## The Database

We will start with the database since it needs to be up and running before our
_TypeScript_ app.

### Create 

Create two kubernetes objects, namely a _Deployment_ and a _Service_ for the
database. In this case we will, for brevity, use a _Deployment_ for our database
with replication set to 1. _Deployments_ are not the best suited. 

The environment variables
- MONGO\_INITDB\_ROOT\_USERNAME
- MONGO\_INITDB\_ROOT\_PASSWORD 

must be placed in a _Secret_ and thus read in the
_Deployment_ also implying that a _Secret_ object must be created using its own
yaml file. At this point the password will be stored directly in the yaml, but
to keep things simple, this is ok for now.

_**Important detail, when specifying this you must access the database
admin. Any other DB won't work!**_

Remember this for your _Application ConfigMap!_


#### Things to remember and head

- The name you give your database _Service_ is the name for which you can access
  it within your _TypeScript_ app.
- How one creates a _Secret_
- How to denote the environment variables in a _Deployment_ / _Pod_ specificiation
  and especially how do you extract _Secrets_ to be used here.
  
### Apply & Verify

Use kubectl to apply the file content with your newly created deployment and
service. You can either choose to have everything in one file or having them in
their own.

Upon having applied the yaml files, inspect the setup using _OpenLens_ and
verify that the _Service_ you created in the chosen namespace is visible and
points to the intended _Deployment_.

## The Node application

### Preparation

As opposed to the previous exercise, the image we want to use resides on
https://gitlab.au.dk, which means that authentication is needed. This is
accomplished by having a _Access Token_ created in our _Preferences_ @
https://gitlab.au.dk. Bare minimum required access rights for the token is that
it must provide _read_ access to _registries_. 


Next create a secret docker registry token, so as to provide access to the
GitLab registry. Remember to place it in the same namespace as the app.

Note that in the below the secret is named _my-registry-credentials_ but can be
any name you chose!

> kubectl create secret docker-registry my-registry-credentials --docker-server=https://registry.gitlab.au.dk --docker-username=TOKEN-NAME --docker-password=TOKEN-PASSWORD --docker-email=YOUR-EMAIL -n YOUR-NAMESPACE

kubectl create secret docker-registry registry-credentials --docker-server=https://registry.gitlab.au.dk --docker-username=glpat-sxzYRCmxoxK7eyWDTRmi --docker-password=glpat-sxzYRCmxoxK7eyWDTRmi --docker-email=201910895@post.au.dk -n holy-namespace



### Getting started

The yaml files you created in the previous lecture for _Deployment_, _Service_
and _Ingress_ are suitable starting points for this exercise.

### Deployment

#### Create

To enable the _Deployment_ to fetch the desired image from AU's GitLab you need
to specify the socalled _pull secret_:

>       imagePullSecrets: 
>            - name: my-registry-credentials

The following environment variables must be set:
- MONGODB\_USERNAME
  - Current value "" 
  - Get value from _Secret_ containing the value for MONGO\_INITDB\_ROOT\_USERNAME
- MONGODB\_PASSWORD
  - Current value ""
  - Get value from _Secret_ containing the value for MONGO\_INITDB\_ROOT\_PASSWORD
- MONGODB\_DBNAME
  - Current value "testdb"
  - Read this value from a ConfigMap -> Must be _admin_ when using _INITDB_ROOT...__
- MONGODB\_HOSTNAME
  - Current value "mongo"
  - Depending on what you have called your service this needs to be adjusted. Add this information to a ConfigMap.
- REL\_URL\_PATH
  - Current value "/"
  - Change it to whatever you want, just not '/' and place the information in a ConfigMap.

#### Things to remember and heed

- 2 of the above environment variables are _Secrets_, as denoted, remember to
  refer to the same _Secret_ created in the DB _Deployment_.
- 3 of the above environment variables are to be read from a ConfigMap. Create a
  ConfigMap with the information described and ensure that the references are
  correct.
- Remember the value / subpath url set in REL\_URL\_PATH. It is to be used in
  the _Ingress_ specification.
  

#### Apply & Verify

Use kubectl to apply the file content with your newly created deployment.

Inspect via _OpenLens_ after having applied the yaml file and verify that the
_Deployment_ you created in the chosen namespace is visible.

Find the running _Pod_ using _OpenLens_ and
- Inspect logs and verify that the app is running and is connected to the DB
- _Port-forward_ the _Pod_ and verify that you can connect to it using a browser
  with the chosen suburlpath.


### Service & Ingress

#### Create 

The _Service_ and _Ingress_ specifications are very much identical to the
exercise in the previous lecture. The primary changes resides in the images
used, the naming and the fact that the _urlpath_ specified must be identical to
the one contained in REL\_URL\_PATH!

#### Things to remember and head
- Naming of labels and references must match perfect!
- URL paths must match

#### Apply 

Use kubectl to apply the file content with your newly created _Service_ and
_Ingress_.


#### Accessing (solved)

>If you have done everything correctly at this point, then opening your browser
at http://mycluster.my:8080/api should direct you to the newly created application via
the _Ingress_ through the _Service_ unto the(a) _Pod_ itself. Where XXXX is
REL\_URL\_PATH.


## Storage test

Having access to the web app add some tasks to it (or do something that saves
data to the DB).

Restart the DB and retrieve the task list. At this point with this particular
configuration, what do you expect to happen and why?


# Training exercise 2 - Storage

## Intro

The solution created in exercise 1 only keeps data saved in the DB temporarily
in that they are lost when the _Pod_ is removed/restarted. A more persistent
solution is therefore desired.

In effect two forms of storage provisioning exists, namely _Static_ and _Dynamic_
provisioning. In this exercise we will focus on the _Dynamic_ part and let the
_Static_ be left for you to explore yourselves.

## Dynamic provisioning

To provide _Dynamic_ provisioning one needs to:

- Create a named _Persistent Volume Claim (PVC)__
- Add a reference to this _Persistent Volume Claim (PVC)_ to your DB deployment


### Persistent Volume Claim

Create a yaml file describing the PVC request as you deem needed. Capacity could
be 2Gi, which would be more than adequate.

Consider the _accessModes_ specifier. Which one is your choice and why?

In this scenario we will forego specififying storage class name. Why do you
think this is so?

#### Things to remember and head
- Proper naming and namespace
- Access mode choice
- Storage needed
- Storage class


### DB Deployment

In your Mongo DB deployment (assuming Mongo) add a volume mount. The _mountPath_
being _/data/db_. Give it an appropriate name.

In the _volumes_ part add the volume mount name and add a reference to the PVC
just created.

#### Things to remember and heed
- What is a volume mount and what does mean / is important
- The _volumes_ section is important why?
  - The PVC specified with associated _claimName_ 

### Apply 

Use kubectl to apply the file containing the _PVC_ as well as the updated
version of the DB _Deployment_.


## Verify

Again verify that you can access the app using http://mycluster/XXXXX, create
tasks and save them.

## Storage test

Restart the DB Pod again and again reflect upon what you recon will happen and
thus what you expect to see.


## Scaling

Task: Scale your DB deployment to 3

Before you carry on, think about what you recon will happen and why!



# Training exercise 3 - StatefulSet

## Intro

Finally the last part will deal with creating a proper persistent storage for
each of the deployed pods. This is achieved using the _kind_ _StatefulSet_,
which facilitates pricisely this. 

Important note: In the previous exercises _Secrets_ were employed to handle DB
user and password, however when using _replicas_ this is significantly more
difficult and timewise not worth it in this exercise. IF _you_ wish to employ
this now or later go read
https://www.mongodb.com/docs/v2.6/tutorial/deploy-replica-set-with-auth/

Otherwise, when reusing/copying the yaml files used in the previous exercise
remove all references to username/password for accessing the DB. 

## Create / Update

### Service - Headless

Creating a _StatefulSet_ is different in the sense that it's desired to be able
to access each _Pod_ or at least it can be prudent as is the case when working
with _mongodb_.

As always one needs a service to access the _Pods_, normally though, a _Service_
has an IP, however in this case one specifies that the _clusterIP_ is to be
_None_. Assuming that the _Service_ name is _mongo_ and that the name of
_StatefulSet_ likewise is _mongo_, each _Pod_ will be accessible namewise in the
network _mongo-X.mongo_, where X is an ordinal in the range 0-2 since the number
of _replicas_ is set to 3.

### StatefulSet

Creating a _StatefulSet_ is simimlar to that of a _Deployment_. The primary
change notation wise, in this simple sense, is merely the addition of
_volumeClaimTemplates_ that tells kubernetes how to facilitate _Persistent
Volume Claims_ to each _Pod_. You simply write a so-called template and k8s
handles the rest.

Since the idea is that the _mongo_ instances are to work in concert, they need
each to be informed of the fact. The _volumeClaimTemplates_ array notationwise
similar to that of the already know _PersistentVolumeClaims_. Therefore reuse
the one from the previous exercise.

Lastly the command to run _mongodb_ must be altered such that _mongo_ **knows** that
it is to work on a _replicaset_.

The command to run _mongo_ is therefore replaced using this command line below:

> mongod --replSet rs0 --bind\_ip\_all

DO NOTE that it has to be an array, with each to these elements on their own
line, otherwise it will not work!!

#### Things to remember and head

- What exacly makes a _Service_ _headless_ and how does this affect accessing
  the underlying _Pods_?
    >- Setting clusterIP == None makes it Headless
  >* Accessing each of the 3 replicas DNS wise
  >* The DNS naming convention is
    ``StatefulSet.name + "-" + "ordinal index" +
    "." + Service.name``
  >* Here
    Replica 1: mongo-0.mongo-service
    Replica 2: mongo-1.mongo-service
    Replica 3: mongo-2.mongo-service

- How is a _volumeClaimTemplates_ different from a _PersistentVolumeClaim_?

  > **_volumeClaimTemplates_** is a template for creating **_PersistentVolumeClaims_** for each _Pod_ in the _StatefulSet_.



### The Node application

In the previous exercise the MongoDB server hostname was _mongo_, this is no
longer the case since we need client access to all 3.

The hostname is therefore, e.g. a list

> mongo-0.mongo:27017,mongo-1.mongo:27017,mongo-1.mongo:27017


### Apply

Use kubectl to apply the files as usual, however having done that one final
action is needed. At this point the mongo servers have now been started with
modified commandline options telling them to work on a replicaset.  However they
are not bound together and thus not working together.

Configuring the main _mongo_ node is therefore required. This can be done with the
following commands.

```
rs.initiate()
var cfg=rs.conf()
cfg.members[0].host="mongo-0.mongo:27017"
rs.reconfig(cfg)
rs.add("mongo-1.mongo:27017")
rs.add("mongo-2.mongo:27017")
```

This is assuming that the _Service_ name is _mongo_ and the container name for
each is likewise _mongo_.

Save the above text to some file (here replica.conf) and run:

> kubectl exec -it -n XXX mongo-0 mongosh < replica.conf

Note that the namespace chosen here is _XXX_, replace it with your own.




### Verify and Inspect

#### Web 

Launch a browser as before and verify that the application works and thus saves
data as expected.


#### Storage test

Encompasses two things:
- Verify that a restart of the diffferent _Pods_ does not mean any loss of data.
- Delete the StatefulSet
  - Are the PVC/PVs still intact?
  - What happens if the StatefulSet is reapplied? Is the data intact?

# Training exercise 4  - MongoDB done right - BONUS

There are numerous challenges getting MongoDB running correctly, when you want
scaling to work. We have addressed some of it, however for a more proven and
sound approach use the _MongoDB Kubernetes Operator_.

This is task where you are on your own! 

Follow the guide(s)
https://www.mongodb.com/docs/kubernetes-operator/v1.18/kind-quick-start/ and
have fun... ;-)

OR you can also go with the _bitnami_ helm install... Up to you!
