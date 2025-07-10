# CKS TIPS


- Code
- Container
- Cluster
- Cloud


# ###########################
# ###########################
# ###########################
# ###########################
# ###########################
# --- CLUSTER SETUP AND HARDENING ---
# ###########################


- CIS benchmark (k8s pdf document)  - https://www.cisecurity.org/benchmark/kubernetes
- CIS-CAT - tool to check your system (Kubernetes only in PRO version ) 
  - RUN - sh ./Assessor-CLI.sh -i -rd /var/www/html/ -nts -rp index



- kube-bench - 

  
- kube-bench run --targets=master | grep kube-controller -A 3 -  show "-A" + 3 lines 
- kube-bench run --targets=master | grep "/var/lib/etcd" -B5

- kube-bench run --targets=master --check 1.2.15 

- also targets can be [master node controlplane etcd policies ] ->  --targets=node,etcd


- kube-bench run --benchmark cis-1.6 



- ./kube-bench --config-dir ./cfg --config ./cfg/config.yaml


- ps -ef | grep kubelet








- Kubernetes Auth Mechanism (set up in kube-apiserver)
  - Static password file - curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123" - Also, note that this approach is deprecated in Kubernetes version 1.19 and is no longer available in later releases.
  - Static token file    
  - Certificates 
  - Identity Service 


- Service Accounts (SA)
  - User SA - for user 
  - "compute" SA  - for tools like Jenkins, Prometheus  and etc 

  


- Certification
  -  get info about cert - openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text 





- Create account for user
  - create CertificateSigningRequest with test.key
  - approve it - kubectl certificate approve NAME 
  - or reject  - kubectl certificate deny NAME





- Kube config 
  - get current context - kubectl config current-context --kubeconfig my-kube-config 
  - switch context -  kubectl config --kubeconfig=/root/my-kube-config use-context research
  - 




Add the my-kube-config file to the KUBECONFIG environment variable.

Open your shell configuration file:
vi ~/.bashrc

Add the following line to export the variable:
export KUBECONFIG=/root/my-kube-config

Apply the Changes:

Reload the shell configuration to apply the changes in the current session:

source ~/.bashrc




# ACCESS TO API SERVER BY KUBE PROXY  - Opens proxy port to API server


- Run kube proxy 
  - kubectl proxy --port=8090 & (default port 8001)
- Curl to localhost
  - curl http://localhost:8090/api
  - curl 127.0.0.1:8090/version

# PORT FORWARD  - Opens port to target deployment pods
- kubectl port-forward pods/my_pod 8005:80 &
  - pod port - 80
  - port on my pc - 8005


# ACCESS TO API SERVER BY IP ADDRESS 
-  curl request - curl -X GET K8S_ADDRESS/api --header "Authorization: Bearer  SECRET_FROM_K8S" --insecure


# 
- kubectl auth can-i delete node 
- kubectl auth can-i create deployments 

kubectl get pods --as dev-user


# Authorization type  - kube-apiserver  --authorization-mode=......

- Node authorizer - Node Authorization is a specific type of authorization mode in Kubernetes 
  that is used to authorize API requests made by kubelets. 
  It is not intended for user authorization.
- RBAC    - role base access control 
- Webhook - allows for custom authorization logic by delegating the authorization decision to an external HTTP service
- AlwaysAllow - allows all requests without any further authorization checks.
- AlwaysDeny - denies all requests without any further authorization checks.





# RBAC

- Get authorization-mode of cluster  - kubectl describe pod kube-apiserver-controlplane -n kube-system
- Check permission of user - kubectl get pods --as dev-user
- create Role - kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
- create RoleBinding - kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user





# ROLE vs CLUSTER ROLE 


Parameter	                Role	                                            ClusterRole

Scope	            Works within a single namespace	               Works across the entire cluster
Resource Access     Access to namespace-scoped resources	       Access to both namespace-scoped and cluster-scoped resources
Resource Types	    Namespace-scoped (e.g., Pods, Services)	       Cluster-scoped (e.g., Nodes, PVs) or Namespace-scoped
Binding Object	    RoleBinding	                                   ClusterRoleBinding




# ROLE BINDING vs CLUSTER ROLE BINDING 

Binding Type	         Binds to	        Scope
RoleBinding	           A  Role	          A specific  namespace
ClusterRoleBinding	   A ClusterRole	  The entire cluster




# Удалить пользователи test из группы dev с помощью kubectl

kubectl get rolebinding -A | grep dev
kubectl edit rolebinding dev-access -n default






# Kubelet Security 

- test kubelet - curl -sk https://localhost:10250/pods



- check Kubelet process  = --config=/var/lib/kubelet/config.yaml

- 10250 - allow full access 
- 10255 - allow unauthenticated read-only access

- --anonymous-auth=false


- Authentication = certificates (recommended) or API Bearer Tokens 
  - Needs to set up ca file in kubelet and kubelet client key/cert in kube-apiserver

- --authorization-mode=Webhook

- --read-only-port=0 - disable it 




# K8S software version 

- K8s components have the same version 
- Only Etcd and CoreDNS have own version 





# Network policy 

Check folder  - [network_policy](network_policy)



- get labels - `k get pods --show-labels`


# Node Metadata ( needs to block it)
- get metadata about node - kubectl exec app -- curl -s http://controlplane:9999

- need to create network policy to block trafik to node metadata 




!!!!!!!!!!!!!!!!!!!

# Audit policy 





Need to create audit policy

Args in kube-apiserver.yaml

- --audit-policy-file=/etc/kubernetes/audit/policy.yaml
- --audit-log-path=/etc/kubernetes/audit/logs/audit.log
- --audit-log-maxsize=5  ---- maximum size in megabytes
- --audit-log-maxbackup=1   
- --audit-log-maxage=20   ----- maximum age of the log entries should not exceed 30 days.



```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  namespaces: ["prod"]
  verbs: ["delete"]
  resources:
  - group: ""
    resources: ["secrets"]
```


```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:

# log Secret resources audits, level Metadata
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]

# log node related audits, level RequestResponse
- level: RequestResponse
  userGroups: ["system:nodes"]

# for everything else don't log anything
- level: None
```



...
volumes:
- name: audit
  hostPath:
  path: /etc/kubernetes/audit-policy.yaml
  type: File

- name: audit-log
  hostPath:
  path: /var/log/kubernetes/audit/
  type: DirectoryOrCreate
-




then mount the volumes:
...
volumeMounts:
- mountPath: /etc/kubernetes/audit-policy.yaml
  name: audit
  readOnly: true
- mountPath: /var/log/kubernetes/audit/
  name: audit-log
  readOnly: false
  and finally configure the hostPath:





# ################
# ################
# ################
# ################
# --- SYSTEM HARDENING ---
# ################




Limit node access

- no public access to worker-node
- only private access to master node (vpn, or something else )
- only public access to "load balancer" (only 443 port )


- user account       - your account 
- superuser account  - root 
- system account     - ssh 
- service account    - http, nginx 

- id   - show info about account
- who  - show user who login 
- last - show last user login 


- userdel / groupdel                
- usermod -s /usr/sbin/nologin himanshi
- useradd -d /opt/sam -s /bin/bash -G admin -u 2328 sam


apt list --installed
apt remove nginx -y
apt install wget -y

systemctl list-units --all
netstat -natp  | grep 9090




# Tracing syscalls


find pods with name start with collector1 - `crictl pods --name collector1`

show container with pod - `crictl ps --pod a61e29997e607`

get all info abount container id - `crictl inspect CONTAINER_ID`

get process -` ps aux | grep collector1-process`

find Syscalls - `strace -p 14079`


pidof etcd

strace -p pid_number 



strace kill -9 1234 - покажет что происходит когда - kill -9 1234



!!!!!! needs to read about syscall 

tracee - help you work with Tracing in vm/k8s
https://github.com/aquasecurity/tracee


Seccomp in Kubernetes 





# AppArmor 




- create rule - rule.yaml 

```yaml
#include <tunables/global>

profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,

  # Deny all file writes.
  deny /** w,
}
```


- add this profile            - `apparmor_parser rule.yaml`   
- check if it was created     - `apparmor_status | grep "k8s-apparmor-example-deny-write"` 

- copy file to all node - `scp app_rule.yaml node01:/root`

- create pod 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor
spec:
  securityContext:
    appArmorProfile: # this 
      type: Localhost
      localhostProfile: k8s-apparmor-example-deny-write
  containers:
  - name: hello
    image: busybox:1.28
    command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
```






# ################
# ################
# ################
# ################
# --- MICROSERVICES AND OTHER ---
# ################



# securityContext 

- check - [security_context](security_context)(security_)




# Admission Controllers 

Check each request to the API server before it’s persisted to etcd, and can 



Example

In kube-apiserver needs add:   --enable-admission-plugins=NamespaceLifecycle,LimitRanger,PodSecurity,...


Enable:

- kubectl run nginx --image nginx -n blue
- Error from server (NotFound): namespaces "blue" not found
- Because kubernetes have NamespaceExists admission controller enabled which rejects requests to namespaces that do not exist.
- ADD - --enable-admission-plugins=NamespaceAutoProvision,..... to kube-apiserver
- NamespaceAutoProvision admission controller - create namespace automatically 

Disable: 


- add --disable-admission-plugins=DefaultStorageClass  to kube-apiserver 



Get all admission-plugins 

ps -ef | grep kube-apiserver | grep admission-plugins






also we can create our own policy and enable it 



!!!!!!!!!!!!!!!!!!!!!!!!!!!

For example - imagepolicywebhook

https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook

https://github.com/moabukar/CKS-Exercises-Certified-Kubernetes-Security-Specialist/blob/main/7-mock-exam-questions/Q10-ImagePolicyWebhook.md






!!!!!!!!!!!!!!!!!!!

# Pod security admission  (PSA) -  check pod security before creating  (MODE)



MODE:
- Warn    -   just warn log 
- Audit   -   Write audit log
- Enforce -  	Policy violations will cause the pod to be rejected.

#  Pod security standards  - base policy for pod  (LEVEL)

LEVEL:
- Privileged - Full access / Unrestricted policy, providing the widest possible level of permissions.
- Baseline   - Minimally restrictive policy which prevents known privilege escalations.
- Restricted - Heavily restricted policy, following current Pod hardening best practices.



To enable Pod Security: 

- pod-security.kubernetes.io/<MODE>: <LEVEL>

- kubectl label ns alpha pod-security.kubernetes.io/warn=baseline

- kubectl label ns beta pod-security.kubernetes.io/enforce=baseline pod-security.kubernetes.io/warn=restricted




```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-user-container
  namespace: k29
spec:
  containers:
    - image: bitnami/nginx:1.21.6
      name: secured-container
      securityContext:                  # add this 
        runAsNonRoot: true              # add this 
        allowPrivilegeEscalation: false # add this 
        capabilities:                   # add this 
          drop: ["ALL"]                 # add this 
        seccompProfile:                 # add this 
          type: RuntimeDefault          # add this 
```










# Open Policy Agent (OPA)

- Gatekeeper 


untrusted-registry.rego

package kubernetes.admission

deny[msg] {
  input.request.kind.kind == "Pod"
  image := input.request.object.spec.containers[_].image
  not startswith(image, "hooli.com/")
  msg := sprintf("image '%v' comes from untrusted registry", [image])
}

allow only pod with hooli.com/ image 


admission webhook "validating-webhook.openpolicyagent.org" denied the request: image 'nginx' comes from untrusted registry




# Policy engine
- Kyverno





# Apiserver Security 

-  Accessible through a NodePort Service - `--kubernetes-service-node-port=31000`





- Only accessible through a ClusterIP Service - `delete kubernetes-service-node-port or set --kubernetes-service-node-port=0`
- check kubernetes service port type - `k get svc`
- delete kubernetes service (it will be recreated) - `k delete svc kubernetes`


# Secrets 


* Create secret - `kubectl create secret generic my-secret --from-literal=DB_PASSWORD=password`
* Create secret - `kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`
* echo -n 'test1' | base64



ETCD in Kubernetes stores data under `/registry/{type}/{namespace}/{name}`. 

read secrets fromk etcd -` ETCDCTL_API=3 etcdctl \
--cert /etc/kubernetes/pki/apiserver-etcd-client.crt \
--key /etc/kubernetes/pki/apiserver-etcd-client.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt get /registry/secrets/team-green/database-access`


- Decode secrets  -` echo Y29uZmlkZW50aWFs | base64 -d`


- if you don't have access to secret - `k -n restricted exec pod1-fd5d64b9c-pcx6q -- cat /etc/secret-volume/password `



````yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-pod
    namespace: default
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: db-secret`
````


# Encrypt  secrets  in etcd 

- https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/ 






# Container Runtime Sandbox 

Use gVisor and RuntimeClass with gVisor 






# kubernetes ingress redirect http to https



```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: world
  namespace: world
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"  # add this !!!!
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  ingressClassName: nginx
  tls:                            # add
  - hosts:                        # add
    - world.universe.mine         # add
    secretName: ingress-tls       # add
  rules:
  - host: "world.universe.mine"
    http:
      paths:
      - path: /europe
        pathType: Prefix
        backend:
          service:
            name: europe
            port:
              number: 80
      - path: /asia
        pathType: Prefix
        backend:
          service:
            name: asia
            port:
              number: 80
```


# ResourceQuota


- kubectl describe quota team-a-resource-quota -n team-a
 

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-resource-quota
  namespace: team-a
spec:
  hard:
    pods: "5"
    requests.cpu: "0.5"
    requests.memory: 500Mi
    limits.cpu: "1"
    limits.memory: 1Gi
```


# Taint and Toleration 

- taint       
  - add    - `kubectl taint nodes node01 team=team-a:NoSchedule`
  - remove - `kubectl taint nodes node01 team=team-a:NoSchedule-`
- toleration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: team-a-pod
  namespace: team-a
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "team"
      operator: "Equal"
      value: "team-a"
      effect: "NoSchedule"

```






# ################
# ################
# ################
# ################
# --- SUPPLY CHAIN SECURITY  ---
# ################



!!!!!!!!!!!!!!!!!!!

# SBOM  
это структурированный список всех компонентов, библиотек, зависимостей и модулей,
которые используются в программном обеспечении. 
Проще говоря, это как «список ингредиентов» для программы.



Зачем нужен SBOM

Безопасность
— Позволяет быстро находить уязвимые компоненты при выходе новых CVE (например, уязвимость в log4j).
— Упрощает аудит безопасности и лицензий.
Соответствие требованиям
— Некоторые стандарты и законы (например, Executive Order 14028 в США) требуют от поставщиков наличие SBOM.
Трассируемость
— Упрощает понимание, какие версии библиотек используются и как они влияют на поведение приложения.




bom generate --image registry.k8s.io/kube-apiserver:v1.31.0 --format json --output /opt/course/18/sbom1.json




Scan image  - `trivy image --format cyclonedx --output /opt/course/18/sbom2.json registry.k8s.io/kube-controller-manager:v1.31.0`
Show vulnerabilities with critical - `trivy sbom /opt/course/18/sbom_check.json --severity CRITICAL`



Scan image - `trivy image nginx:1.16.1-alpine `





syft — генератор SBOM

- curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
- syft docker.io/kodekloud/webapp-color:latest -o spdx > /root/webapp-spdx.sbom
- syft docker.io/kodekloud/webapp-color:latest -o cyclonedx-json > /root/webapp-sbom.json



grype — сканер уязвимостей
- curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin
- grype sbom:/root/webapp-sbom.json -o json > /root/grype-report.json





# KubeLinter 
- статический анализ  yaml




# kubesec

- статический анализ  yaml
run as 
- binary 
- docker contaider 
- kubelet plugin 
- Admission controller (kubesec-webhook)




# Whitelisting allowed registries (our images )

We can use 
- Admission Controller
- OPA (Open Policy Agent ) or Policy engine 
- ImagePolicyWebhook








# ################
# ################
# ################
# ################
# --- MONITORING  LOGGING AND RUNTIME SECURITY ---
# ################



!!!!!!!!!!!!!!!!!!!

Falco  - 


- falco -U | grep httpd - show log and show container_id 
- falco --list | grep container - show all available fields like -  %evt.time,%container.id,%container.name,%user.name


- crictl ps -id container_id

- crictl pods -id pod_id - show pod by id 
- crictl pods -id cab6dafd045d5 - give you service name 



- create rule in /etc/falco/falco_rules.local.yaml

```yaml
- rule: Read Sensitive File
  desc: Alert when someone tries to read a sensitive file like /etc/shadow
  condition: open_read and fd.name=/etc/shadow
  output: "Sensitive file read (user=%user.name command=%proc.cmdline)"
  priority: ERROR
  tags: [filesystem, sensitive]
```



- чекает что происходит в кластере 


🛡 Зачем нужен Falco

Назначение	                                Примеры
🐞 Обнаружение нарушений безопасности	Установка exec в контейнере, запуск bash
📜 Аудит и соответствие требованиям	    Уведомления о доступе к чувствительным файлам
🔥 Реагирование на инциденты	        Алерты в Prometheus, Slack, SIEM
👀 Мониторинг runtime-поведений	        Нестандартные подключения, работа с сокетами




!!!!!!!!!!!!!!!!!!!




# Immutable Root FileSystem


```yaml
        securityContext:                  # add
          readOnlyRootFilesystem: true    # add
```




# Make it impossible to podman exec , docker exec or kubectl exec into the container using bash


```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get -y install curl
ENV URL https://google.com/this-will-fail?secret-token=
RUN rm /usr/bin/bash  # !!!!!!!!!!!!!!!!!!!!!!!!!!!!
CMD ["sh", "-c", "curl --head $URL$TOKEN"]
```
