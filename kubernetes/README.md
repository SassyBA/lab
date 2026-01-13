https://greatamerican.udemy.com/course/kubernetes-masterclass-for-beginners

## Setup (Mac + Podman + MiniKube)

### Export the corporate Root GAIG CA from macOS

Keychain Access -> System Roots -> Root GAIG CA -> Export as root-gaig-ca.pem

### Install the Corporate CA into the Podman VM

Podman on macOS runs all containers inside a Linux VM.
This VM must trust your corporate CA before any container can pull images.

1. Base64‑encode the PEM file on macOS: `base64 -i ~/Desktop/root-gaig-ca.pem > root-gaig-ca.pem.b64`
2. Create the trust directory inside the Podman VM: `podman machine ssh -- sudo mkdir -p /etc/pki/ca-trust/source/anchors`
3. Copy the certificate into the VM using a base64 pipe: `cat root-gaig-ca.pem.b64 | podman machine ssh -- \
"base64 -d | sudo tee /etc/pki/ca-trust/source/anchors/root-gaig-ca.pem >/dev/null"`
4. Update the VM’s trust store: `podman machine ssh -- sudo update-ca-trust`
5. Restart the Podman VM: `podman machine stop` `podman machine start`

### Start an Ubuntu Container and Install the CA Inside It

Each new container starts with a clean trust store.
You must add the corporate CA inside the container as well.

1. Start a fresh Ubuntu container: 
    ```podman run -it \
      -v ~/.kube:/root/.kube \
      ubuntu
    ```
2. Create the directory for custom CAs: `mkdir -p /usr/local/share/ca-certificates`
3. On mac: `cat ~/Desktop/root-gaig-ca.pem` -> copy PEM block
4. Inside the container: `cat > /usr/local/share/ca-certificates/root-gaig-ca.crt`
5. Paste the certificate, then press Ctrl+D.
6. Install CA tools:
    ```
    apt-get update
    apt-get install -y ca-certificates
    ```
7. Update the container’s trust store: `update-ca-certificates`

### Install kubectl Inside the Container

```
apt-get install -y curl
curl -LO https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/bin/
kubectl version --client
```

### Minikube on macOS ARM Without Docker Desktop (QEMU Driver)

1. `brew install minikube`
2. `brew install qemu`
3. `minikube start --driver=qemu`
