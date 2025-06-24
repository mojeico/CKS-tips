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
  - download - https://github.com/aquasecurity/kube-bench
  - run kube-bench - ./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml




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






# Node Metadata ( needs to block it)
- get metadata about node - kubectl exec app -- curl -s http://controlplane:9999




# Audit policy 






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



pidof etcd

strace -p pid_number 

!!!!!! needs to read about syscall 

tracee - help you work with Tracing in vm/k8s
https://github.com/aquasecurity/tracee


Seccomp in Kubernetes 





# AppArmor 

!!!! needs to read about AppArmor

and AppArmor in Kubernetes 














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








# Pod security admission  (PSA) -  check pod security before creating  (MODE)

#  Pod security standards  - base policy for pod  (LEVEL)

MODE:
- Warn    -   just warn log 
- Audit   -   Write audit log
- Enforce -  	Policy violations will cause the pod to be rejected.


LEVEL:
- Privileged - Full access / Unrestricted policy, providing the widest possible level of permissions.
- Baseline   - Minimally restrictive policy which prevents known privilege escalations.
- Restricted - Heavily restricted policy, following current Pod hardening best practices.



To enable Pod Security: 

- pod-security.kubernetes.io/<MODE>: <LEVEL>

- kubectl label ns alpha pod-security.kubernetes.io/warn=baseline

- kubectl label ns beta pod-security.kubernetes.io/enforce=baseline pod-security.kubernetes.io/warn=restricted


TO create your own rule - you need to create AdmissionConfiguration 





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
































