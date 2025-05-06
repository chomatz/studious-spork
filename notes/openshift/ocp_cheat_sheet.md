ocp cheat sheet
===============

resources
---------

the below commands inspect the current namespace - *add `-n <NAMESPACE_IDENTIFIER>` to target a different namespace from the current one*

### setup a terminal on a running pod
`oc rsh <POD_IDENTIFIER>`

### get logs from pods
`oc logs <POD_IDENTIFIER>`

### inspect resources

- show all
`oc get all`

- show secrets - *does not appear in `oc get all`*
`oc get secret`

- display secret
`oc extract secret/<SECRET_IDENTIFIER> --to=-`

### cleanup

- delete resources defined by manifest files
`oc delete -f </PATH/TO/FILE>`
`oc delete -f </PATH/TO/FILE1> -f </PATH/TO/FILE2> -f </PATH/TO/FILEX>`
`oc delete -f .`

htpasswd authentication
-----------------------

- create the htpasswd file
`htpasswd -c -B -b <PATH/TO/HTPASSWD/FILE> <USERNAME> <PASSWORD>`

- add additional htpasswd file entries
`htpasswd -b <PATH/TO/HTPASSWD/FILE> <USERNAME> <PASSWORD>`

- create the htpasswd secret
`oc create secret generic <SECRET_IDENTIFIER> --from-file htpasswd=<PATH/TO/HTPASSWD/FILE> -n openshift-config`

- assign a user to a role
`oc adm policy add-cluster-role-to-user <ROLE_IDENTIFIER> <USER_IDENTIFIER>`

- export oauth configuration
`oc get oauth cluster -o yaml > </PATH/TO/FILE>`

- edit oauth cluster configuration
```
apiVersion: config.openshift.io/v1
kind: OAuth
...output omitted...
spec:
  identityProviders:
  - ldap:
<OUTPUT_OMITTED>
    type: LDAP
  - name: <PROVIDER_PREFIX>
    type: HTPasswd
    mappingMethod: claim
    htpasswd:
      fileData:
        name: <SECRET_IDENTIFIER>
```

- extract htpasswd secret
`oc extract secret/<SECRET_IDENTIFIER> -n openshift-config --to </TARGET/EXTRACTION/PATH> --confirm`

- add/update htpasswd file entries
`htpasswd -b <PATH/TO/HTPASSWD/FILE> <USERNAME> <PASSWORD>`

- remove htpasswd file entries
`htpasswd -D <PATH/TO/HTPASSWD/FILE> <USERNAME>`

- update htpasswd secret
`oc set data secret<SECRET_IDENTIFIER> -n openshift-config --from-file htpasswd=<PATH/TO/HTPASSWD/FILE>`

- delete identities as needed
`oc delete identity <PROVIDER_PREFIX>:<USERNAME>`

- delete users as needed
`oc delete user <USERNAME>`

rbac
----

### default roles

- **admin** - administrative access on a project
- **basic-user** - read-only access
- **cluster-admin** - administrative access on entire cluster
- **cluster-status** - access cluster status information
- **cluster-reade** - view most of the objects but cannot modify them
- **edit** - modify common application resources on the project
- **self-provisioner** - can create own projects
- **view** - view project resources

### rbac help

- list cluster policy options
`oc adm policy --help`

- list project policy options
`oc policy --help`

- manage groups
`oc adm groups --help`

- disable self provisioning
`oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth`

deployments
-----------

- restart a deployment
`oc rollout restart deployment/<DEPLOYMENT_NAME>`

manifests
---------

### apply manifests

- single file
`oc apply -f </PATH/TO/FILE>`

- multiple files
`oc apply -f </PATH/TO/FILE1> -f </PATH/TO/FILE2> -f </PATH/TO/FILEX>`
`oc apply -f .`

- do not make changes - validate only
`oc apply -f </PATH/TO/FILE> --dry-run=server --validate=true`

### view changes

- single file
`oc diff -f </PATH/TO/FILE>`

- multiple files
`oc diff -f </PATH/TO/FILE1> -f </PATH/TO/FILE2> -f </PATH/TO/FILEX>`
`oc diff -f .`

routes
------

### expose service

`oc expose svc <SERVICE_IDENTIFIER> --hostname <APPLICATION_URL>`

### ssl

- create ssl secret
`oc create secret tls <SECRET_IDENTIFIER> --cert </PATH/TO/CERTIFICATE> --KEY </PATH/TO/PRIVATE/KEY>`

- create an encrypted edge route
`oc create route edge <ROUTE_IDENTIFIER> --service <SERVICE_IDENTIFIER> --hostname <APPLICATION_URL> --key </PATH/TO/PRIVATE/KEY> --cert </PATH/TO/CERTIFICATE>`

- create a passthrough route
`oc create route passthrough <ROUTE_IDENTIFIER> --service <SERVICE_IDENTIFIER> --port <PORT_IDENTIFIER> --hostname <APPLICATION_URL>`

- add a tls secret to a service
`oc annotate service <SERVICE_IDENTIFIER> service.beta.openshift.io/serving-cert-secret-name=<SECRET_IDENTIFIER>`

- sample nginx deployment patch file
```
spec:
  template:
    spec:
      containers:
        - name: server
          volumeMounts:
            - name: nginx_secret
              mountPath: /etc/pki/nginx/
      volumes:
        - name: nginx_secret
          secret:
            defaultMode: 420
            secretName: nginx_secret
            items:
              - key: tls.crt
                path: server.crt
              - key: tls.key
                path: private/server.key
```

- patch secret mountpoint on a deployment
`oc patch deployment <DEPLOYMENT_IDENTIFIER> --patch-file </PATH/TO/FILE>`

- create a ca-bundle configmap
`oc create configmap <CONFIGMAP_IDENTIFIER>`
`oc annotate configmap <CONFIGMAP_IDENTIFIER> service.beta.openshift.io/inject-cabundle=true`

helm
----

### inspect helm resources

- display helm releases
`helm list`

- display chart information
`helm show chart <CHART_IDENTIFIER>`

- display default chart values
`helm show values <CHART_IDENTIFIER>`

### manage repositories

- display helm repos
`helm repo list`

- add a help repo
`helm repo add <REPOSITORY_IDENTIFIER> <REPOSITORY_URL>`

### manage charts

- search for charts
`helm search repo`

- search for charts and display al;l versions
`helm search repo --versions`

- test installation
`helm install <RELEASE_IDENTIFIER> <CHART_IDENTIFIER> --dry-run --values </PATH/TO/VALUES/FILE>`

- upgrade release
`helm upgrade <RELEASE_IDENTIFIER> <CHART_IDENTIFIER> --version <VERSION_IDENTIFIER> --values </PATH/TO/VALUES/FILE>`

### rollback releases
`helm history <RELEASE_IDENTIFIER>`
`helm rollback <RELEASE_IDENTIFIER> <REVISION_IDENTIFIER>`

network policies
----------------

- create a policy
`oc create -n <NAMESPACE_IDENTIFIER> -f </PATH/TO/FILE>`

- view network plocies
`oc get networkpolicy -n <NAMESPACE_IDENTIFIER>`

- add a network label onto a namespace
`oc label namespace <NAMESPACE_IDENTIFIER> network=<NETWORK_LABEL>`

- sample policy
```
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: <POLICY_IDENTIFIER>
spec:
  podSelector:
    matchLabels:
      deployment: <DEPLOYMENT_IDENTIFIER>
    ingress:
      - from:
        - namespaceSelector:
            matchLabels:
              network: <NAMESPACE_LABEL>
          podSelector:
            matchLabels:
              deployment: <DEPLOYMENT_IDENTIFIER>
        ports:
        - port: <PORT_IDENTIFIER>
          protocol: <TCP|UDP>
...
```

templates
---------

- list templates
`oc get templates -n openshift`

- get template details
`oc describe template <TEMPLATE_IDENTIFIER> -n openshift`

- get template parameters
`oc process --parameters <TEMPLATE_IDENTIFIER> -n openshift`

- deploy app from template while specifying parameters
`oc new-app --template=<TEMPLATE_IDENTIFIER> -p <PARAM1>=<VALUE1> -p <PARAM2>=<VALUE2> -p <PARAMX>=<VALUEX>`

- use `oc process` to generate a manifest and use `oc apply` to deploy the generated manifest
` oc process <TEMPLATE_IDENTIFIER> -p <PARAM1>=<VALUE1> -p <PARAM2>=<VALUE2> -p <PARAMX>=<VALUEX> | oc apply -f -`
