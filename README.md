# Encrypting Kubernetes Secrets with Sealed Secrets

### This Collection of sealed secrets will be used together with ArgoCD App of Apps pattern during cluster bootstrapping

---

## Quoted from GitOps Fundamentals course

The truth is that there is no single accepted practice for how secrets are managed with GitOps. If you already have a solid solution in place such as HashiCorp vault, then it would make sense to use that even though technically it is against GitOps practices.

If you are starting from scratch on a new project, there are ways to store secrets in Git so that you can manage them using GitOps principles as well. It goes without saying that you should never ever commit raw secrets in Git.

Argo CD can be used with any secret solution that you already have deployed.

All solutions that handle secrets using Git are storing them in an encrypted form. This means that you can get the best of both worlds. Secrets can be managed with GitOps, and they can also be placed in a secure manner in any Git repository (even public repositories).

As an example, we will use the Bitnamic Sealed secrets controller to encrypt secrets before storing them in Git and decrypt them before passing them to the application.

## Bitnami Sealed Secrets

Sealed secrets are just an extension built on top of Kubernetes native secrets. This means that after the encryption/decryption takes place, all secrets function as plain Kubernetes secrets, and this is how your application should access them. If you don’t like how plain Kubernetes secrets work, then you need to find an alternative security mechanism.

The Bitnami Sealed secrets controller is a Kubernetes controller you install on the cluster that performs a single task. It converts sealed secrets (that can be committed to Git) to plain secrets (that can be used in your application).

Sealed Secrets consists of two components:

- Client-side CLI tool to encrypt secrets and create sealed secrets
- Server-side controller used to decrypt sealed secrets and create secrets

## Creating a sealed secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mimir-bucket-secret
data:
  AWS_ACCESS_KEY_ID: FAKEACCESSKEY
  AWS_SECRET_ACCESS_KEY: FAKESECRETKEY
```

Save the file with any name that you prefer, for me it would be mimir-bucket-secret.yaml.
do not forget to also encode your secrets with base64 first.

## Sealing the secrets

```bash
cat mimir-bucket-secret.yaml | kubeseal --format yaml > mimir-bucket-secret-sealed.yaml
```

In case you installing Bitnami Sealed Secrets using a non default namespace and release name during installation via Helm, you need no include some switches to make it run without any issues.

    --controller-name string           Name of sealed-secrets controller. (default "sealed-secrets-controller")
    --controller-namespace string      Namespace of sealed-secrets controller. (default "kube-system")

```bash
cat secret.yaml | kubeseal \
    --controller-namespace kube-system \
    --controller-name sealed-secrets \
    --format yaml \
    > sealed-secret.yaml
```
