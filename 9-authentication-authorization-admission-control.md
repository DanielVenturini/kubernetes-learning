# Authentication, Authorization, Admission Control

Every API request reaching the API server has to go through several control stages before being accepted by the server and acted upon. In this chapter, we will learn about the Authentication, Authorization and Admission Control stages of the Kubernetes API access control.

---

Before a command be executed or a resource be accessed, the request must be processed in three different layers, or access control stages. These access control stages are:

- **Authentication**: authenticates an users, that is, this access is validated as a Kubernetes user.
- **Authorization**: authorizes the API requests, that is, this stage verifies whether the authenticated user has the authorization to access to API.
- **Admission Control**: this stage validates/modifies the user requests.

### Authentication

Kubernetes does not have any kind object called user or username or something like that. However, Kubernetes can use usernames for the [Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) phase of the API access control and to request logging as well. There is two kind of Kubernetes users:

- **Normal Users**: these users are not added to cluster through an API call, but they are created by an administrator distributing private keys, by a Keystone or Google Accounts or a file with a list of usernames and passwords.
- **Service Accounts**: these accounts allow in-cluster processes to communicate with the API server to perform various operations. Most of the Services Accounts are automatically created via the API server, but these can be manually created. The Service Accounts are tied to a particular Namespace.

Kubernetes can also support [anonymous request](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests) and [user impersonation](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation). The last method is useful when administration is troubleshooting authorization polices. For authentication, Kubernetes uses a series of [authentication modules](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#authentication-strategies):

- **X509 Client Certifies**: the client certificate authorization is done through a file that contains one or more certificate authorities (CA), using the `--client-ca-file=ca.file.ext` command. Then, CA authorizes the clients certificates presented by users to the API server.

- **Static Token File**: we can pass a file with a pre-defined token through the `--token-auth-file=token.file.ext` command. When a token is changed, the whole API server must be changed too.

- **Bootstrap Tokens**: tokens used for bootstrapping new Kubernetes cluster.

- **Service Account Tokens**: these are an automatically enabled authenticators that are attached to Pods using the Service Account [Admission Controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/).

- **OpenID Connect Tokens**: these tokens provide a way to connect with OAuth2, such as Azure Active Directory, Salesforce and Google.

- **Webhook Token Authentication**: with this method, verification of bearer tokens can be offloaded to a remote service.

- **Authenticating Proxy**: allows for the programming of additional authentication logic.

We can enable multiple authenticators ways. We should enable at least two methods: the service account tokens authenticator and one of the user authenticators.

### Authorization

After an user gets access to the API server, the user can send API requests. These requests are analyzed by Kubernetes via [authorization modules](https://kubernetes.io/docs/reference/access-authn-authz/authorization/), that allow or deny the request. The request is applied to policies that check the *user*, *group*, *extra*, *Resource*, *Namespace* and another informations. **A bunch of authorizers**, in sequence, check the request and all of them can approve of deny the request.

There are several different ways to authorize requests:

- **Node**: the [node authorization](https://kubernetes.io/docs/reference/access-authn-authz/node/) authorizes API requests made by kubelets to read/write a lot of Kubernetes objects.

- **Attribute-Based Access Control (ABAC)**: the [ABAC authorizer](https://kubernetes.io/docs/reference/access-authn-authz/abac/), it can specifies in an authorization file the which user can access which resource and the mode. In the next example, the user `student` can only read pods from the namespace `lfs148`.

  ```yaml
  apiVersion: abac.authorization.kubernetes.io/v1beta1
  kind: Policy
  spec:
    user: student
    namespace: lfs158
    resource: pods
    readonly: 'true'
  ```
  
  To enable ABAC mode, we must start the API server with the `--authorization-mode=ABAC`.
  
- **Webhook**: the [webhok mode](https://kubernetes.io/docs/reference/access-authn-authz/webhook/) provides a way to grant access to the API for third-part services.

##### **Role-Based Access Control (RBAC)**

Using [RBAC mode](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) to authorize the API requests, we can regulate the access to computers or network resources based on the Roles of individual users. Roles can specify the resource access operation, such as **create**, **get**, **update** etc. There are two kinds of Roles, the **Role**, which one grants access to resources within a specific Namespace, and **ClusterRole**, which grants the same permission as Roles does, but its scope is cluster-wide. The following is an example of a Role:

  ```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: lfs158
  name: pod-reader
rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
  ```

The previous manifest defines a `pod-reader` role that can `get`, `watch` and `list` Pods in namespace `lfs158`.

After a Role was created, we can bind this role to users through a **RoleBinding** object. There are two kinds of RoleBindings:

- **ClusterRoleBinding**: it allows us to grant access to resources at a cluster-level and to all Namespaces.

- **RoleBinding**: allows us **to bind users** to the same namespace **as a Role** and we cloud also refer a ClusterRole in RoleBinding granting permissions to Namespace resources defined in the ClusterRole.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-read-access
  namespace: lfs158
subjects:
  - kind: User
    name: student
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

The above manifest creates a RoleBinding that binds the User `student` to the Role `pod-reader`. Now, the user `student` can only get, watch and list all the Pods in the `lfs158` namespace.

To enable RBAC mode, the API server must be started with the `--authorization-mode=RBAC` option.

### Admission Control

[Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) are used to specify some kinds of control policies to deny/allow API requests in container privileges, resource quotas etc. These policies are forced using different admissions controller, such as `ResourceQuota`, `DefaultStorageClass`, `AlwaysPullImages` etc. If any of the controllers deny the requests, the entire request is rejected.

Admission controllers may *validate* or *mutate* -- or both -- the objects they admit. For example, the ResourceQuota mutates/changes its object, the storage.

To enable admission controllers, we must specify them in the API start up (`#1`) and we can see the controllers that are enabled by default (`#2`):

```bash
kube-apiserver --enable-admission-plugins=NamespaceLifecycle,ResourceQuota,...	#1
kube-apiserver -h | grep enable-admission-plugins	#2
```

There are also a bunch of available custom plugins in [Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/).