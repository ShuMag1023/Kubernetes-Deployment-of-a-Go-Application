# Kubernetes-Deployment-of-a-Go-Application
Let's set up a Kubernetes deployment for a Go application. This project will involve creating a Go application, containerizing it using Docker, creating Kubernetes manifests for deployment, and using Ansible to automate the deployment process.



Let's set up a Kubernetes deployment for a Go application. This project will involve creating a Go application, containerizing it using Docker, creating Kubernetes manifests for deployment, and using Ansible to automate the deployment process.

### Project Outline

1. **Setup GitHub Repository:**
   - Create a GitHub repository for a simple Go application.
   - Include a `Dockerfile` and Kubernetes manifests.

2. **Write Dockerfile:**
   - Create a `Dockerfile` to containerize the Go application.

3. **Create Kubernetes Manifests:**
   - Write YAML files for the Kubernetes deployment and service.

4. **Ansible Playbook for Deployment:**
   - Write an Ansible playbook to automate the deployment to a Kubernetes cluster.

### Detailed Steps

#### Step 1: Setup GitHub Repository

1. Create a new repository on GitHub (e.g., `go-k8s-app`).
2. Initialize a simple Go application in the repository.

**Sample Go Application:**

- `main.go`:
    ```go
    package main

    import (
        "fmt"
        "net/http"
    )

    func helloHandler(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, Kubernetes!")
    }

    func main() {
        http.HandleFunc("/", helloHandler)
        http.ListenAndServe(":8080", nil)
    }
    ```

- `go.mod`:
    ```go
    module github.com/your-username/go-k8s-app

    go 1.16
    ```

#### Step 2: Write Dockerfile

Create a `Dockerfile` in the root of your repository.

**Dockerfile:**
```dockerfile
# Use the official Golang image as a build environment
FROM golang:1.16 as build

# Set the working directory inside the container
WORKDIR /app

# Copy the go.mod and go.sum files and download dependencies
COPY go.mod go.sum ./
RUN go mod download

# Copy the rest of the application code
COPY . .

# Build the Go application
RUN go build -o main .

# Use a minimal base image to run the application
FROM alpine:latest

# Set the working directory inside the container
WORKDIR /root/

# Copy the built binary from the build environment
COPY --from=build /app/main .

# Expose the application port
EXPOSE 8080

# Start the application
CMD ["./main"]
```

#### Step 3: Create Kubernetes Manifests

Create a directory `k8s` in your repository and add the following YAML files.

**Deployment YAML (`k8s/deployment.yaml`):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: go-app
  template:
    metadata:
      labels:
        app: go-app
    spec:
      containers:
      - name: go-app
        image: your-docker-repo/go-app:latest
        ports:
        - containerPort: 8080
```

**Service YAML (`k8s/service.yaml`):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: go-app-service
spec:
  selector:
    app: go-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

#### Step 4: Ansible Playbook for Deployment

Create an Ansible playbook (`deploy.yml`) to automate the deployment process.

**Ansible Playbook (`deploy.yml`):**
```yaml
---
- hosts: k8s
  vars:
    repo_url: "https://github.com/your-username/go-k8s-app.git"
    repo_dest: "/tmp/go-k8s-app"
    docker_image: "your-docker-repo/go-app"
    k8s_namespace: "default"

  tasks:
    - name: Install required packages
      apt:
        name: "{{ item }}"
        state: present
      with_items:
        - git
        - docker.io
        - kubectl

    - name: Clone the repository
      git:
        repo: "{{ repo_url }}"
        dest: "{{ repo_dest }}"
        version: HEAD

    - name: Build the Docker image
      command: docker build -t {{ docker_image }} .
      args:
        chdir: "{{ repo_dest }}"

    - name: Push the Docker image to registry
      command: docker push {{ docker_image }}

    - name: Apply Kubernetes deployment
      command: kubectl apply -f deployment.yaml --namespace={{ k8s_namespace }}
      args:
        chdir: "{{ repo_dest }}/k8s"

    - name: Apply Kubernetes service
      command: kubectl apply -f service.yaml --namespace={{ k8s_namespace }}
      args:
        chdir: "{{ repo_dest }}/k8s"
```

#### Step 5: Define Inventory File

Create an inventory file (`hosts.ini`) to define the target server(s).

**Inventory File (`hosts.ini`):**
```ini
[k8s]
your-k8s-master-ip ansible_user=your-ssh-username ansible_ssh_private_key_file=/path/to/your/private/key
```

#### Step 6: Run the Ansible Playbook

Execute the Ansible playbook to deploy the Go application to your Kubernetes cluster.

**Run the Playbook:**
```sh
ansible-playbook -i hosts.ini deploy.yml
```

### Conclusion

By following these steps, you will have deployed a Go application on Kubernetes using Docker for containerization and Ansible for automation. This project provides practical experience with containerization, orchestration, and deployment automation, which are essential skills in modern DevOps practices.
