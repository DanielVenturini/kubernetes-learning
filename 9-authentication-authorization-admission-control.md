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