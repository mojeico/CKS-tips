# CKS TIPS


- Code
- Container
- Cluster
- Cloud




# Cluster Setup and Hardering 


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




# ACCESS TO API SERVER BY KUBE PROXY 


- Run kube proxy 
  - kubectl proxy --port=8090 &
- Curl to localhost
  - curl http://localhost:8090/api


# ACCESS TO API SERVER BY IP ADDRESS 
-  curl request - curl -X GET K8S_ADDRESS/api --header "Authorization: Bearer  SECRET_FROM_K8S" --insecure




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
























































































