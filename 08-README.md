# k8s-rbac-setup
rbac setup in eks clusters


#######Creating Clusterroles,Roles,clusterrolebinding,rolebinding for providing eks cluster access to perform necessary actions####
#########################What is the use of k8s RBAC#################################
#Kubernetes RBAC is a powerful security feature that allows administrators to control who can access the 
#Kubernetes API and what actions they can perform. You can use it to implement the principle of “least privilege,” which means that 
#users should have the minimum levels of access necessary to perform their tasks
################################# END #################################

###Below config is the base configuration for aws-auth ConfigMap which allows worker nodes with InstanceRoleto join the EKS control plane###
#$kubectl edit cm aws-auth

apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::111111111:role/worker-node-instance-role
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
####to add additional access for existing Roles/Users, you can configure like the below###  
#$kubectl edit cm aws-auth -o yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::111111111:role/worker-node-instance-role
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
    - userarn: arn:aws:iam::111111111:role/ssm-role
      username: eks-write-role
  mapUsers: |
    - userarn: arn:aws:iam::111111111:user/rajuuser
      username: rajuuser

####Create an RBAC Role to namespace#####
#role.yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: developv                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     
  name: eks-full-access
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
####Apply the role:####
#$kubectl apply -f role.yaml

####Create RoleBinding for Role####
#rolebinding.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: full-access-role-for-eks
  namespace: develop
subjects:
- kind: User
  name: rajuuser
  apiGroup: rbac.authorization.k8s.io
roleRef:                                  ##We can Ref role from EKS Role which we created the above###
  kind: Role
  name: eks-full-access                 
  apiGroup: rbac.authorization.k8s.io

####Apply the RoleBinding####
#$kubectl apply -f rolebinding.yaml

###Note###
#####At this point if you assume role arn:aws:iam::111111111:role/rajuuser and create kubeconfig for that role you will have full admin access to develop namespace#####

###Manage access using centralised ClusterRole we have to create clusterrole and clusterrolebindings like the below.###
# clusterrole.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cr_full_access
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]

###ClusterRole and ClusterRoleBinding###
###In case you want to grant permissions for your IAM role in all namespaces, you can simply use ClusterRoleBinding as follows####
# clusterrolebinding.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: full_access_cluster_role_binding
subjects:
- kind: User
  name: rajuuser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cr_full_access
  apiGroup: rbac.authorization.k8s.io

###Clusterrole and Custerrolebinding for more resources###
# clusterrole.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cr_full_access
rules:
- apiGroups: ["*"]
  resources:
  - services
  - pods
  verbs: 
  - get
  - list
  - watch
  - delete
- apiGroups: ["*"]
  resources:
  - namespaces
  - pods/logs
  - nodes
  - events
  - services
  verbs: 
  - get
  - list
  - watch
  - patch  
- apiGroups: 
  - app
  - extentions
  resources:
  - deployment
  verbs: 
  - get
  - list
  - watch
  - patch
  - delete
- apiGroups: 
  - extentions
  resources:
  - '*'
  verbs: 
  - get
  - list
  - watch  
- apiGroups: 
  - apps
  resources:
  - '*'
  verbs: 
  - get
  - list
  - watch 

# clusterrolebinding.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: full_access_cluster_role_binding
roleRef:
  kind: ClusterRole
  name: cr_full_access
  apiGroup: rbac.authorization.k8s.io  
subjects:
- kind: User
  name: rajuuser
  apiGroup: rbac.authorization.k8s.io

##################END##########################