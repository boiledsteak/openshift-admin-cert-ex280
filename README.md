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

Kustomize patches!