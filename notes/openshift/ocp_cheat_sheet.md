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

service accounts
----------------

- create service account
`oc create sa <USER_IDENTIFIER> -n <NAMESPACE_IDENTIFIER>`

- add role to service account
`oc adm policy add-role-to-user <ROLE_IDENTIFIER> system:serviceaccount:<NAMESPACE_IDENTIFIER>:<USER_IDENTIFIER>`

- configure scc to run as any uid
`oc adm policy add-scc-to-user anyuid -z <USER_IDENTIFIER> -n <NAMESPACE_IDENTIFIER>`

- configure prvileged scc
`oc adm policy add-scc-to-user privileged -z <USER_IDENTIFIER> -n <NAMESPACE_IDENTIFIER>`

- assign service account to deployment
`oc set serviceaccount deployment/<DEPLOYMENT_IDENTIFIER> <USER_IDENTIFIER> -n <NAMESPACE_IDENTIFIER>`

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

- edit oauth from gui
**Administration** \> **Cluster Settings** \> **Configuration** \> **Oauth** \> **Identity pvoviders** \> **Add** \> **HTPasswd**

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
- **cluster-reader** - view most of the objects but cannot modify them
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

### create a service

`oc expose deployment <DEPLOYMENT_IDENTIFIER> --name <SERVICE_IDENTIFIER> --port <PORT_IDENTIFIER>`

### expose a service

`oc expose svc <SERVICE_IDENTIFIER> --hostname <APPLICATION_URL>`

### ssl

- create ssl secret
`oc create secret tls <SECRET_IDENTIFIER> --cert </PATH/TO/CERTIFICATE> --key </PATH/TO/PRIVATE/KEY>`

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

- search for charts and display all versions
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

limit range
-----------

- sample template
```
apiVersion: v1
kind: LimitRange
metadata:
  name: <LIMITRANGE_IDENTIFIER>
  namespace: <NAMESPACE_IDENTIFIER>
spec:
  limits:
  - type: <Container/Pod>
    default:
      cpu: 200m
      memory: 1Gi
    defaultRequest:
      cpu: 100m
      memory: 1Gi
    max:
      cpu: 300m
      memory: 2Gi
    min:
      cpu: 50m
      memory: 128Mi
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
`oc process <TEMPLATE_IDENTIFIER> -p <PARAM1>=<VALUE1> -p <PARAM2>=<VALUE2> -p <PARAMX>=<VALUEX> | oc apply -f -`

self-service
------------

### create a self-service template

- create project template
`oc adm create-bootstrap-project-template -o yaml > </PATH/TO/FILE>`

- create a limitrange in a namespace
**ocp console** \> **Administration** \> **LimitRanges** \> **Create LimitRange**

- create a quota in a namespace
**ocp console** \> **Administration** \> **ResouceQuotas** \> **Create ResourceQuota**

- append limitrange and quota to template
`oc get limitrange,quota -n <NAMESPACE_IDENTIFIER> -o yaml >> </PATH/TO/FILE>`

- edit template file
  - apply the following changes to the *subjects* key in the *admin* role binding
    - change the *kind* key to *Group*
    - change the *name* key to *<GROUP_IDENTIFIER>*
    - change the *namespace* to *${PROJECT_NAME}*
    - move the *LimitRange* section after the *RoleBinding* definition
    - remove the following keys from the *LimitRange* and *Template* definitions
      - creationTimestamp
      - resourceVersion
      - uid

- create template
`oc apply -f </PATH/TO/FILE> -n openshift-config`

- set template as default for new projects
`oc edit projects.config.openshift.io cluster`
```
apiVersion: config.openshift.io/v1
kind: Project
metadata:
...output omitted...
  name: cluster
...output omitted...
spec:
  projectRequestTemplate:
    name: <TEMPLATE_IDENTIFIER>
```

- wait for pod deployment
`watch oc get pod -n openshift-apiserver`

### modify clusterrolebinding self-provisioners

- edit self-provisioners
`oc edit clusterrolebinding self-provisioners`

- under subjects \> name set name to \<GROUP_IDENTIFIER\>
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
...output omitted...
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: self-provisioner
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: <GROUP_IDENTIFIER>
```

- ensure that the self-provisioner changes do not get rewritten
`oc annotate clusterrolebinding/self-provisioners --overwrite rbac.authorization.kubernetes.io/autoupdate=false`
