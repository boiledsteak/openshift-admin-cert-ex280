# openshift-admin-cert-ex280
my notes on ex280

link to redhat learning DO280 course
https://rol.redhat.com/rol/app/courses/do280-4.22/pages/pr01s02

---
create new project

`oc new-project my-project`

---
add rolebinding to user

`oc adm policy add-role-to-user edit fm_tma`

---

add clusterRoleBinding to user

`oc adm policy add-cluster-role-to-user cluster-admin fm_tma`

---

show console link

`oc whoami --show-console`

---

login via api server

`oc login -u fm_tmah https://api.lab.example.com:6443`

---

view kubeconfig

`oc config view --raw`

or 

`cat ~/.kube/config`

---

view current branch

`git branch`

---

Kustomize!

```
myapp/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── production/
    │   ├── kustomization.yaml
    │   └── route.yaml
    └── staging/
        └── kustomization.yaml
```

the base kuztomization would look like this:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

production's kuztomization would look like this:


```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# Include the shared base resources
resources:
  - ../../base

# Override replica count
replicas:
  - name: myapp
    count: 2

# Apply staging-specific configuration
patches:
  - path: route.yaml
```

---

Kustomize secretGenerator and configMapGenerator

when modifying a secret or configmap, the changes usually dont apply until you restart the deployment

secretGenerator and configMapGenerator helps by creating `<secret-name>-<hash>` or  `<configMap-name>-<hash>` then adjust the deployment.yaml to point to these new objects. This way, the deployment will auto restart since its pointing to a new secret or configMap

```
kind: Kustomization
resources:
- database
- exoplanets
secretGenerator:
- name: db-secrets
  literals:
    - DB_ADMIN_PASSWORD=postgres
    - DB_NAME=database
    - DB_PASSWORD=password
    - DB_USER=user
configMapGenerator:
- name: db-config
  literals:
    - DB_HOST=database
    - DB_PORT=5432
```

so this will generate a secret in the database and exoplanets folder

then generate a configMap in the database and exoplanets folder

---

ocp templates

a very simple and lightweight way to add variables to k8 yamls. but far less featured than helm charts

ie.

```yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: nginx-template
  annotations:
    description: Simple Nginx application

parameters:
  - name: APP_NAME
    description: Application name
    value: nginx

  - name: REPLICAS
    description: Number of replicas
    value: "2"

objects:
  # Deployment
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: ${APP_NAME}
    spec:
      replicas: ${{REPLICAS}}
      selector:
        matchLabels:
          app: ${APP_NAME}
      template:
        metadata:
          labels:
            app: ${APP_NAME}
        spec:
          containers:
            - name: nginx
              image: nginx:latest
              ports:
                - containerPort: 80

  # Service
  - apiVersion: v1
    kind: Service
    metadata:
      name: ${APP_NAME}
    spec:
      selector:
        app: ${APP_NAME}
      ports:
        - port: 80
          targetPort: 80
```

but this shit can't even just `oc apply -f`. Need to run `oc process -f nginx-template.yaml` then it generates the yamls INDIVIDUALLY. Need to `oc apply` them afterwards

---

HTPasswd

a way to auth with openshift using a file that contains username + hashed pw

htpasswd originated from the Apache HTTP Server ecosystem.

Traditionally, it was used to:
- Create username/password files.
- Store hashed passwords.
- Authenticate users accessing password-protected web resources using HTTP Basic Authentication.
OpenShift simply reuses the same HTPasswd file format

the htpasswd file can hold several users

1. run htpasswd command to generate the file

```shell
htpasswd ./location/htpasswd username
```

2. go to GUI, cluster settings, search for OAuth

---

openshift identity

1 user can auth with several OAuth methods. Hence 1 user object can have many identity object that links to different OAuth types

this is a keycloak identity

```yaml

apiVersion: user.openshift.io/v1
kind: Identity
metadata:
  name: keycloak:john
providerName: keycloak
providerUserName: john
user:
  name: john
  uid: 12345678-1234-1234-1234-123456789abc
extra:
  email: john@example.com
  name: John Doe

```

this is htpasswd identity

```yaml
apiVersion: user.openshift.io/v1
kind: Identity
metadata:
  name: htpasswd_provider:john
providerName: htpasswd_provider
providerUserName: john
user:
  name: john
  uid: 12345678-1234-1234-1234-123456789abc
```

---

Deleting user

should delete user object, identity object, and from identity provider side. If it's htpasswd OAuth then its the htpasswd file itself. If it's keycloack OAuth then its on keycloak side

---

