# ConfigMaps and Secrets

While developing and deploying applications, it's almost ever required to use runtime parameters like configuration details, permissions, passwords, tokens an so on. Kubernetes allow us to create environment variables through **ConfigMap API** and **Secret API**.

---

### ConfigMaps

When we want to set configuration details, such as environment variables, to be used in container images, we must use [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/). We can set non-confidential data in key-value pairs and reference it into the image template. We can create ConfigMaps from literal values, from configuration files, from one or more files or directories.

ConfigMap is designed to store small chunks of information, that is, informations that do not hold more than 1MiB of data. If a data is larger than 1MiB, it must be stored into a Volume.

Most of Kubernetes objects have a required `spec` field. However, ConfigMap has as a required fields the `data` and `binaryData` fields in the metadata file. Both are optional, but the `data` aims with a UTF-8 key-value pair whereas `binaryData` aims with binary data as base64-encoded strings.

Both ConfigMap and Secret can be created through an yaml file or directly by a CLI. The following example contains a CLI and an yaml example:

```bash
kubectl create configmap config-information --from-literal=course="Introduction to Kubernetes" --from-literal=platform=edX
kubectl get configmap config-information -o yaml	# output is similar to the next example
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-information
data:
  course: "Introduction to Kubernetes"
  platform: "edX"
binaryData:
  username: "ZGFuaWVsdmVudHVyaW5pCg=="
  password: "bm90aGluZyBoZXJlISBtb3ZlIGFsb25nCg=="
```

There is another way to create a ConfigMap. In this way, we have an external file that contains, for each row, a key-value pair entry. The following example is a file called `permission-reset.properties`.

```bash
permission=read-only
default-prefix="user-${USERNAME}"
resetCount=3
```

Then, we can create via the following command:

```bash
kubectl create configmap permission-reset --from-file=~/git/permission-reset.properties
kubectl describe configmap permission-reset
```

However, this is a bit of wired to create a ConfigMap from an external file.

---

### Secrets

[Secret](https://kubernetes.io/docs/concepts/configuration/secret/) is similar to ConfigMap, but this object must be used when sensitive information is shared, such as a password or a token. Once we can store a token in a Secret Kubernetes' object, we can load it into the environment and we do not need to save it in a text file that may be expose for anyone in, for example, GitHub. Secret is also a key-value pair.

> Secrets are stored as plain text into etcd, so, anyone that can access etcd is allowed to see the password. So, the etcd and API access must be limited by the administrators.

We can create a Secret through the CLI or by an yaml file:

```bash
kubectl create secret generic passwords --from-literal=git=123lab
```
When creating through a file, all secrets within `data` field must be encoded in base64, whereas data within `stringData` not. Values within `stringData` is encoded when the Secret is created:
```yaml
apiVersion: v1
kind: Secret
metadata:
  namespace: venturini
  name: passwords
type: Opaque
data:
  git: dGhlcmUgaXMgbm90aGluZyBoZXJlIGFnYWluISBtb3ZlIGFsb25nCg==
stringData:
  gmail: itwillbeencodedsoon
```

We can also create a Secret from a plain text file, as in ConfigMap. However, all content must be previously encoded.

```bash
echo "mysecret" | base64 > mysecret.txt	# create an encoded file
kubectl create secret generic mysecret --from-file=mysecret.txt
kubectl get secret mysecret
```

The `get` command does not show the content of the secret, just the keys.

---

### Insert into a Pod

The next step is to attach those information inside the Pod. Once the ConfigMap/Secret is available, we can [configure the Pod and make those information available](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) inside the Pod. There are two ways to retrieve the key-value pair of data inside a Pod:

- **As environment variables**: we can set inside a Pod metadata **a bunch of specific values** or **the whole ConfigMap/Secret** values. The following example sets the [whole ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables) as environment variables inside a Pod:

  ```yaml
  containers:
    - name: to-fill-unique-pod
      image: danielventurini/to-fill:1.0
      envFrom:
        - configMapRef:
            name: config-information
        - secretRef:
            name: passwords
  ```

  The following example **sets [specific ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data) variables** as variables inside a Pod:

  ```yaml
  containers:
    - name: to-fill-unique-pod
      image: danielventurini/to-fill:1.0
      env:
        - name: COURSE
          valueFrom:
            configMapKeyRef:
              name: config-information
              key: course
        - name: GMAIL
          valueFrom:
            secretKeyRef:
              name: passwords
              key: gmail
  ```

  Kubernetes loads the container's environment with a variable `COURSE` that refers from the `course` key from `config-information` ConfigMap. Then, we can verify using the following commands:

  ```bash
  kubectl exec -it podname -- /bin/bash
  echo $GMAIL
  echo $COURSE
  ```

- **Volumes**: we can also mount the [ConfigMap as a Volume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume) inside a Pod. For each key in the ConfigMap, a file is created, in the mount path, and the content of the file (named as the key) is the key's value.

  ```yaml
  containers:
    - name: to-fill-unique-container
      image: danielventurini/to-fill:1.0
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
        - name: secret-volume
          mountPath: /etc/secret
  volumes:
    - name: config-volume
    configMap:
        name: config-information
  - name: secret-volume
      secret:
        secretName: passwords	# secretName: Are you sure k8s?
  ```
  
  Then, we can run the following commands to verify the content of the path `/etc/config`:

  ```bash
  kubectl exec -it podname -- /bin/bash
  ls /etc/config			# There is a file for each ConfigMap keys
  cat /etc/config/course	# contains the value of course key
  cat /etc/secret/git
  ```
  

---

You should now be able to:

- Discuss configuration management for applications in Kubernetes using ConfigMaps.
- Share sensitive data (such as passwords) using Secrets.