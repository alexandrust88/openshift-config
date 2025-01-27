1. deploy cluster
2. apply kustomizations from ocp-admin
3. wait for operators to deploy
4. apply argocd + devworkspaces
apply odf path with kustomize

labs order : ocp admin 

1. olm : https://github.com/devops-gang/openshift-dev/blob/main/lab05.mdx 
2. helm https://github.com/devops-gang/openshift-dev/blob/main/lab08.mdx
3. lab : resource contraints https://github.com/devops-gang/openshift-dev/blob/main/lab09.mdx
4. parksmap : https://github.com/devops-gang/openshift-dev/blob/main/lab10.mdx
5. network policies : https://github.com/devops-gang/openshift-dev/blob/main/lab11.mdx
6. 12-18 e argocd 
7.
8.
9.
10.







# openshift-config

curl --silent --location --remote-name "https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize/v5.3.0/kustomize_kustomize.v5.3.0_linux_amd64" && chmod a+x kustomize_kustomize.v5.3.0_linux_amd64 && sudo mv kustomize_kustomize.v5.3.0_linux_amd64 /usr/local/bin/kustomize
notes







install tools 

[\[Title\](ocp-admin/oauth/base/oauth.yaml)](ocp-admin/oauth/base/oauth.yaml)


[Title](ocp-admin/oauth/base/oauth.yaml)

https://github.com/ocpdude/ocp-argocd/blob/main/openshift-cluster/redcloud/apiserver/base/kustomization.yaml

oc apply -k config/


http://gitea-ci.apps.training123.ocptraininglabs.org/

http://nexus-ci.apps.training123.ocptraininglabs.org/


export OCP_USER=1988
export GITEA_USER=adminadmin
export GITEA_PASSWORD=adminadmin
export GITEA_URL=http://gitea-ci.apps.training123.ocptraininglabs.org

https://github.com/redhat-gpte-devopsautomation/coffee-shop-source.git

oc new-app --name=barista \
   openshift/ubi8-openjdk-11:1.3~${GITEA_URL}/adminadmin/coffee-shop-source.git \
   --context-dir=barista \
   --source-secret=gitea-secret \
   --build-env=MAVEN_MIRROR_URL=http://nexus.ci.svc:8081/repository/maven-all-public/ \
   --labels=app.kubernetes.io/part-of=coffee-shop,app.openshift.io/runtime=quarkus \
   -n ${OCP_USER}-dev-coffeeshop


oc new-app --name=coffee-shop \
   openshift/ubi8-openjdk-11:1.3~${GITEA_URL}/adminadmin/coffee-shop-source.git \
   --context-dir=coffee-shop \
   --source-secret=gitea-secret \

   --labels=app.kubernetes.io/part-of=coffee-shop,app.openshift.io/runtime=quarkus \
   -n ${OCP_USER}-dev-coffeeshop


dupa ce devspaces e creat-- > go to route

create workspace: from git : https://raw.githubusercontent.com/RedHat-EMEA-SSA-Team/end-to-end-developer-workshop/rhds/devfile.yaml 

https://raw.githubusercontent.com/RedHat-EMEA-SSA-Team/end-to-end-developer-workshop/rhds/devfile.yaml

https://github.com/redhat-developer/devspaces-demo/blob/main/change-default-ide.sh
https://che.eclipseprojects.io/2021/10/12/@mario.loriedo-devfile-v2-and-the-devworkspace-operator-p1.html

This is a script which has some configuration modules for a fresh and clean OpenShift installation. If you've been using the Single Node OpenShift installation, you need some persistent volumes and you need to make the internal registry able to store your images. 

This is what the script does right now:

- It is setting up Kubevirt Hostpath Provisioner for storage
- Configuring the internal registry to be able to store images
- Installing some operators (namely gitops, serverless and pipelines, postgresql and strimzi) for demo purposes
- Setting up a 'ci' namespace with nexus and Gitea ready to use
- Setting up a htpasswd identity provider for 2 users: admin and devel. admin is cluster-admin
- Configuring openshift console to show links and other resources for demo purposes
- Installing and configuring OpenShift ServiceMesh for demo

## Usage
In order to use this script, you first need to be logged into OpenShift as a user with cluster-admin role. This is typically the `kubeadmin` user after a fresh installation.

The following modules are available:
- console
- storage
- registry
- operators
- ci
- users
- mesh

The `ci` module does only need to have a user context (devel or developer).

The typical workflow after installing a OpenShift SNO from this page is:
- Create the OpenShift SNO
- `./setup.sh storage`
- `./setup.sh registry`
- `./setup.sh users`


You could also let openshift-config decide what to install in your freshly installed OpenShift cluster by specifying an environment:
- sno: installs everything necessary (console, operators, storage, registry, users)
- crc: installs only necessary (console, operators, users)
- aws: installs only necessary (console, operators, users)
- all: like sno


### storage
```bash
$> ./setup.sh storage
```

This installs the CSI kubevirt hostpath provisioner into the target OpenShift installation. It also creates a StorageClass which will then be used for all PVs.

### registry
```bash
$> ./setup.sh registry 
```

This switches the internal registry to "Managed" and binds it to a PV called pv0001. For this module, you must be logged in as kube:admin in your cluster. 

### operators
```bash
$> ./setup.sh operators
```

This installs several operators. Please make sure to be logged into the OpenShift instance as cluster-admin. And you already should have storage available.

### ci
```bash
$> ./setup.sh ci
```

This installs a Nexus and Gitea into a newly created namespace called ci. It also clones 3 repositories from github.com into Gitea. After that it prints out the access to both instances. By default, we are using

- Nexus: admin/admin123
- Gitea: gitea/openshift

### users
```bash
$> ./setup.sh users 
```
This adds a HTPasswd identity provider to the cluster with 2 users in it:
- admin/admin123 -> cluster-admin
- devel/devel -> normal user

You need to be logged into OpenShift with cluster-admin rights.

### console
```bash
$> ./setup.sh console
```
Installs some links to the openshift console and makes sure, developer view contains additional links to resources: pod, service, pvc, route

### mesh
```bash
$> ./setup.sh mesh
```
Installs Red Hat OpenShift ServiceMesh and adds the grumpycat application to the mesh.

This can be uninstalled by executing 
```bash
$> oc apply -k config/mesh
```

If you want to add other namespaces to the mesh, please edit the file `config/mesh/sm-members.yaml`. Please also note, that - in order to let the mesh inject the sidecar - you need to add the following annotation to the `Deployment` of a - well - deployment:

```yaml
spec:
  template:
    metadata:
      annotations:  
        sidecar.istio.io/inject: 'true'
```
