# Activity 10 – Concepts Explained (Beginner Friendly)

**Topic:** Ingress NGINX – Multi-Pod "Fan Out" routing in Kubernetes

---

## 1. The Big Picture (a story)

Imagine a **big office building** (your Kubernetes cluster).

- Inside, there are **employees** doing work → these are your **Pods** (running apps).
- Employees move around, get replaced, and change desks (Pod IP addresses change all the time).
- Visitors from outside need to reach the right employee.

To make that work, we need:

| Real-life thing | Kubernetes thing | Job |
|---|---|---|
| Employee | **Pod** | Runs your app |
| Extension number (stable, never changes) | **Service** | Gives Pods a stable address inside the cluster |
| Receptionist at the front door | **Ingress** | Looks at the visitor's request and sends them to the right Service |
| The receptionist's "brain/employee" that actually does the work | **Ingress Controller (NGINX)** | The program that carries out the Ingress rules |

---

## 2. Each concept in simple words

### Pod
The smallest unit in Kubernetes. It is a wrapper around one container (your app).
In this activity: `app1` and `app2` — each one just replies with text (`hello world 1` / `hello world 2`).

> Problem: a Pod's IP address changes if it restarts. You can't rely on it.

### Service
A **stable name + IP** in front of Pods. It finds Pods using **labels** (like name tags).
The Service type used here is **ClusterIP** → reachable **only inside** the cluster. Outsiders can't reach it directly.

```
Service app1-service  --(finds Pods with label app: app1)-->  Pod app1
```

### Ingress
A **set of routing rules** written in YAML. It says things like:

- "If the URL is `chandan.com/v1`, send it to `app1-service`."
- "If the URL is `chandan.com/v2`, send it to `app2-service`."

**Important:** An Ingress is *only rules*. By itself it does nothing.

### Ingress Controller (NGINX)
The actual program (a web server/reverse proxy, here **NGINX**) that reads the Ingress rules and does the routing. No controller = the rules are ignored.
In Minikube you turn it on with one command: `minikube addons enable ingress`.

### Ingress Class
Tells Kubernetes **which controller** should handle this Ingress. We write `ingressClassName: nginx` so the NGINX controller picks it up.

### Host / Domain name
`chandan.com` is a fake domain. Since it doesn't exist on the real internet, we tell our own computer where it lives by editing the **hosts file**:
`127.0.0.1  chandan.com` → "when I type chandan.com, go to my own machine."

### Minikube tunnel
Minikube runs Kubernetes in a small VM/container on your laptop. `minikube tunnel` opens a path from your laptop (127.0.0.1) into the cluster so your browser can reach the Ingress controller.

---

## 3. What is "Fan Out"?

**Fan out** = **one domain, many paths, each path goes to a different service** (like a hand fan spreading out from one point).

```
                               ┌──> /v1 ──> app1-service ──> Pod app1  ("hello world 1")
 User ──> chandan.com ──> Ingress ┤
                               └──> /v2 ──> app2-service ──> Pod app2  ("hello world 2")
```

Why it's useful: you only need **one domain name (and one entry point)** for many microservices, instead of a separate address for each.

---

## 4. Full request journey (step by step)

1. You open `http://chandan.com/v1` in the browser.
2. Your computer checks the **hosts file** → `chandan.com` = `127.0.0.1`.
3. `minikube tunnel` carries the request into the cluster to the **NGINX Ingress Controller**.
4. NGINX checks the **Ingress rules**: path `/v1` → `app1-service`.
5. The **Service** forwards the request to the **Pod** `app1` on port 8000.
6. The Pod replies `hello world 1`, and the answer travels back to your browser.

Changing the URL to `/v2` follows the same path but ends at `app2` → `hello world 2`.

---

## 5. Key YAML ideas (so the files make sense)

- **`labels`** (on Pod) and **`selector`** (on Service) must match. This is how a Service finds its Pod.
- **`port` / `targetPort`** on the Service: `targetPort` must equal the port the app listens on (**8000** here).
- **`pathType: Prefix`** means "anything starting with `/v1`".
- **`ingressClassName: nginx`** links the Ingress to the NGINX controller.

---

## 6. Common beginner mistakes

| Mistake | Result |
|---|---|
| Label and selector don't match | Service has no endpoints → `503` / no response |
| Wrong `targetPort` | Connection fails |
| Forgot to enable the ingress addon | Ingress gets no address, nothing works |
| Forgot hosts file entry | Browser says "site can't be reached" |
| Closed the `minikube tunnel` window | Site stops working |
| Edited hosts file without Administrator rights | Can't save the file |

---

## 7. One-sentence summary

> **Pods** run the apps, **Services** give them stable internal addresses, and an **Ingress** (executed by the **NGINX Ingress Controller**) lets users reach different Services through **one domain** using different **URL paths**.
