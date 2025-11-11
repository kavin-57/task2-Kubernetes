Task 2: Kubernetes Deployment

Overview
Deployment of the Task Manager Spring Boot application to Kubernetes with MongoDB persistence and Kubernetes pod execution.

Features
•	✅ Spring Boot REST API deployed to Kubernetes
•	✅ MongoDB running in separate pod with persistent storage
•	✅ Kubernetes Pod Execution - Tasks run in temporary pods instead of locally
•	✅ RBAC Configuration for secure Kubernetes API access
•	✅ NodePort Service for external access from host machine
•	✅ Persistent Volume Claims for data persistence
•	✅ Environment Variable configuration for MongoDB connection

Prerequisites
•	Docker Desktop with Kubernetes enabled
•	kubectl CLI
•	Maven
•	Java 17

Quick Start
1. Build Application
cd taskrunner
mvn clean package
docker build -t taskrunner:latest .

2. Deploy to Kubernetes
# Apply all Kubernetes manifests
kubectl apply -f k8s/mongo-pvc.yaml
kubectl apply -f k8s/mongo.yaml
kubectl apply -f k8s/app-rbac.yaml
kubectl apply -f k8s/taskrunner.yaml
# Verify pods are running
kubectl get pods --watch

3. Access Application
# Port forwarding
kubectl port-forward deployment/taskrunner 8080:8080
# Or use NodePort directly
kubectl get services
# Access via: http://localhost:30080/api/tasks
Kubernetes Manifests
MongoDB with Persistent Storage
File: k8s/mongo-pvc.yaml
yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
File: k8s/mongo.yaml
•	MongoDB deployment and service
•	Persistent volume mount
•	Authentication enabled
Application Deployment
File: k8s/taskrunner.yaml
•	Spring Boot application deployment
•	NodePort service (port 30080)
•	Environment variables for MongoDB connection
•	Resource limits and requests
RBAC Configuration
File: k8s/app-rbac.yaml
•	Service account for the application
•	Cluster role with pod creation permissions
•	Cluster role binding
Key Feature: Kubernetes Pod Execution
The main requirement for Task 2 is implemented in TaskService.java:
java
public TaskExecution runCommandForTask(String taskId) throws Exception {
    // Creates temporary Kubernetes pod instead of running locally
    try (KubernetesClient client = new DefaultKubernetesClient()) {
        Pod pod = new PodBuilder()
            .withNewMetadata()
            .withName(podName)
            .endMetadata()
            .withNewSpec()
            .addNewContainer()
            .withName("runner")
            .withImage("busybox:latest")  // Uses busybox as recommended
            .withCommand("sh", "-c", command)
            .endContainer()
            .endSpec()
            .build();
        
        // Create pod, wait for completion, capture logs, delete pod
        client.pods().inNamespace(namespace).create(pod);
        // ... execution logic
    }
}
API Endpoints
All endpoints from Task 1 are available:
•	GET /api/tasks - Get all tasks or single task with ?id=
•	PUT /api/tasks - Create/update task (with command validation)
•	DELETE /api/tasks/{id} - Delete task
•	GET /api/tasks/search?q=name - Search tasks by name
•	PUT /api/tasks/{id}/run - Execute task in Kubernetes pod
Verification Steps
1. Verify Kubernetes Deployment
# Check all resources
kubectl get all

# Check persistent storage
kubectl get pvc,pv

# Check RBAC
kubectl get serviceaccount,clusterrole,clusterrolebinding
2. Test Data Persistence
# Restart MongoDB pod
kubectl delete pod -l app=mongo

# Verify data persists
kubectl get pods -l app=mongo --watch
curl http://localhost:8080/api/tasks
3. Test Kubernetes Pod Execution
# Create a task
curl -X PUT -H "Content-Type: application/json" -d '{
    "id": "k8s-test",
    "name": "Kubernetes Test",
    "owner": "Developer",
    "command": "echo Hello from Kubernetes Pod"
}' http://localhost:8080/api/tasks

# Execute the task (creates Kubernetes pod)
curl -X PUT http://localhost:8080/api/tasks/k8s-test/run

# Watch for temporary pods
kubectl get pods --watch
Troubleshooting
Common Issues
1.	Kubernetes not running: Enable Kubernetes in Docker Desktop
2.	Image pull errors: Build image with docker build -t taskrunner:latest .
3.	RBAC permissions: Verify service account has proper roles
4.	MongoDB connection: Check environment variables and service discovery
Debug Commands
# Check pod logs
kubectl logs deployment/taskrunner

# Check pod details
kubectl describe pod <pod-name>

# Check service endpoints
kubectl describe service taskrunner

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
Environment Variables
The application uses these environment variables:
•	SPRING_DATA_MONGODB_URI: MongoDB connection string
•	K8S_NAMESPACE: Kubernetes namespace (default: default)

Docker Image
Multi-stage Docker build:
•	Stage 1: Maven build with Java 17
•	Stage 2: JRE runtime with optimized image
Compliance with Requirements
•	✅ Application deployed to Kubernetes
•	✅ MongoDB in separate pod with persistent storage
•	✅ Environment variables for configuration
•	✅ Endpoints accessible from host machine
•	✅ Task execution creates Kubernetes pods (not local)
•	✅ Proof via kubectl commands and curl tests

Screenshots
See screenshots/task2/ directory for:
•	Running pods in Kubernetes
•	Services with NodePort
•	Persistent volume claims
•	Task execution creating temporary pods
•	API endpoint accessibility proof

