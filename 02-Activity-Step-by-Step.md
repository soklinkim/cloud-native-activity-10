# Activity 10 – Step-by-Step Instructions (Windows)

**Goal:** Reach `http://chandan.com/v1` → `hello world 1` and `http://chandan.com/v2` → `hello world 2` using two Pods, two Services, and one NGINX Ingress.

## Prerequisites

- Docker Desktop (running), `minikube`, and `kubectl` installed.
- Use **PowerShell** for the commands below.
- Check the installs:

```powershell
minikube version
kubectl version --client
docker --version
```

## Image used

We use **`hashicorp/http-echo`**. It is a tiny web server that returns whatever text you give it, and you can choose the port. That lets us listen on **port 8000** and return a different message per Pod. (You may use another echo server or your own image; just keep the port consistent.)

## Create a working folder

```powershell
cd "d:\AUPP\Fall 2026\Cloud Native\Activity 10"
mkdir k8s
cd k8s
```

---

## Step 1 – (Activity 1) Choose the image
Using `hashicorp/http-echo` (see above). Nothing to run.

## Step 2 – Create the Pods YAML (`pods.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app1
  labels:
    app: app1
spec:
  containers:
    - name: app1
      image: hashicorp/http-echo
      args:
        - "-listen=:8000"
        - "-text=hello world 1"
      ports:
        - containerPort: 8000
---
apiVersion: v1
kind: Pod
metadata:
  name: app2
  labels:
    app: app2
spec:
  containers:
    - name: app2
      image: hashicorp/http-echo
      args:
        - "-listen=:8000"
        - "-text=hello world 2"
      ports:
        - containerPort: 8000
```

## Step 3 – Create the Services YAML (`services.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app1-service
spec:
  type: ClusterIP
  selector:
    app: app1
  ports:
    - port: 8000
      targetPort: 8000
---
apiVersion: v1
kind: Service
metadata:
  name: app2-service
spec:
  type: ClusterIP
  selector:
    app: app2
  ports:
    - port: 8000
      targetPort: 8000
```

## Step 4 – Create the Ingress YAML (`ingress.yaml`)  📸 *Screenshot of this file*

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: chandan-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: chandan.com
      http:
        paths:
          - path: /v1
            pathType: Prefix
            backend:
              service:
                name: app1-service
                port:
                  number: 8000
          - path: /v2
            pathType: Prefix
            backend:
              service:
                name: app2-service
                port:
                  number: 8000
```

> `http-echo` returns the same text for any path, so no URL rewrite is needed.

## Step 5 – Start Minikube and the dashboard

```powershell
minikube start --driver=docker
```

Open a **new PowerShell window** (it stays running) and run:

```powershell
minikube dashboard
```

The dashboard opens in your browser. Leave that window open.

## Step 6 – Enable the Ingress controller  📸 *Command screenshot*

```powershell
minikube addons enable ingress
```

Check the controller is running (wait until `STATUS` is `Running`):

```powershell
kubectl get pods -n ingress-nginx
```

## Step 7 – Add the domain to the Windows hosts file

1. Open **Notepad as Administrator** (search Notepad → right-click → *Run as administrator*).
2. File → Open → `C:\Windows\System32\drivers\etc\hosts` (set the file filter to *All Files*).
3. Add this line at the bottom, then save:

```
127.0.0.1   chandan.com
```

*(Mac: `sudo nano /etc/hosts` and add the same line.)*

## Step 8 – Apply the Pods

```powershell
kubectl apply -f pods.yaml
kubectl get pods
```

Wait until both show `Running`.
📸 **Screenshot from the Minikube Dashboard → Workloads → Pods** (shows app1 and app2).

## Step 9 – Apply the Services, then the Ingress

```powershell
kubectl apply -f services.yaml
kubectl get svc
```

📸 **Screenshot from the Minikube Dashboard → Service → Services** (shows app1-service and app2-service).

Then apply the Ingress:

```powershell
kubectl apply -f ingress.yaml
kubectl get ingress
```

Wait a minute until the `ADDRESS` column is filled.

## Step 10 – Display all endpoints  📸 *Command + output screenshot*

```powershell
kubectl get endpoints
kubectl get all
kubectl get ingress
```

Each service (`app1-service`, `app2-service`) should show an IP like `10.244.x.x:8000`. If it shows `<none>`, the label and selector don't match (see Troubleshooting).

## Step 11 – Start the Minikube tunnel

Open **another PowerShell window as Administrator** and run:

```powershell
minikube tunnel
```

Keep this window open for the whole test. (If it asks for a password, accept it.)

## Step 12 – Access both apps  📸 *2 screenshots showing the URL*

In a browser (or Postman, or `curl`):

| URL | Expected response |
|---|---|
| `http://chandan.com/v1` | `hello world 1` |
| `http://chandan.com/v2` | `hello world 2` |

Command-line check:

```powershell
curl http://chandan.com/v1
curl http://chandan.com/v2
```

Take one screenshot for each, with the **URL bar visible**, so it is clear they reach two different Pods.

---

## Screenshot checklist (for submission)

- [ ] Ingress YAML file (Step 4)
- [ ] `minikube addons enable ingress` command (Step 6)
- [ ] Dashboard – Pods (Step 8)
- [ ] Dashboard – Services (Step 9)
- [ ] `kubectl get endpoints` command and output (Step 10)
- [ ] Browser/Postman – `/v1` with URL visible (Step 12)
- [ ] Browser/Postman – `/v2` with URL visible (Step 12)

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `chandan.com` can't be reached | Check the hosts line (`127.0.0.1 chandan.com`) and that `minikube tunnel` is still running |
| Page shows `503` / `404` from nginx | `kubectl get endpoints`. If empty, make sure the Pod `labels` match the Service `selector` |
| Ingress has no ADDRESS | Wait a minute, and check `kubectl get pods -n ingress-nginx` is Running |
| `kubectl apply` of ingress fails with a webhook error | Wait 1–2 minutes for the ingress controller to be ready, then re-apply |
| Can't save the hosts file | Open Notepad as Administrator |
| Browser keeps redirecting or caches | Try an incognito window, or use `curl` |
| Port 80 in use | Close apps that use port 80 (IIS, Skype, etc.) |

## Cleanup (optional, after you finish)

```powershell
kubectl delete -f ingress.yaml
kubectl delete -f services.yaml
kubectl delete -f pods.yaml
minikube stop
```

Press `Ctrl+C` in the tunnel and dashboard windows to stop them.
