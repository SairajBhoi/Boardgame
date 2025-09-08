# BoardgameListingWebApp

## Description

**Board Game Database Full-Stack Web Application.**
This web application displays lists of board games and their reviews. While anyone can view the board game lists and reviews, they are required to log in to add/ edit the board games and their reviews. The 'users' have the authority to add board games to the list and add reviews, and the 'managers' have the authority to edit/ delete the reviews on top of the authorities of users.  

## Technologies

- Java
- Spring Boot
- Amazon Web Services(AWS) EC2
- Thymeleaf
- Thymeleaf Fragments
- HTML5
- CSS
- JavaScript
- Spring MVC
- JDBC
- H2 Database Engine (In-memory)
- JUnit test framework
- Spring Security
- Twitter Bootstrap
- Maven

## Features

- Full-Stack Application
- UI components created with Thymeleaf and styled with Twitter Bootstrap
- Authentication and authorization using Spring Security
  - Authentication by allowing the users to authenticate with a username and password
  - Authorization by granting different permissions based on the roles (non-members, users, and managers)
- Different roles (non-members, users, and managers) with varying levels of permissions
  - Non-members only can see the boardgame lists and reviews
  - Users can add board games and write reviews
  - Managers can edit and delete the reviews
- Deployed the application on AWS EC2
- JUnit test framework for unit testing
- Spring MVC best practices to segregate views, controllers, and database packages
- JDBC for database connectivity and interaction
- CRUD (Create, Read, Update, Delete) operations for managing data in the database
- Schema.sql file to customize the schema and input initial data
- Thymeleaf Fragments to reduce redundancy of repeating HTML elements (head, footer, navigation)

## How to Run

1. Clone the repository
2. Open the project in your IDE of choice
3. Run the application
4. To use initial user data, use the following credentials.
  - username: bugs    |     password: bunny (user role)
  - username: daffy   |     password: duck  (manager role)
5. You can also sign-up as a new user and customize your role to play with the application! 😊
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# EKS Cluster Setup
- Defined in `k8/eks/cluster.yaml`
- Created using: `eksctl create cluster -f k8/eks/cluster.yaml`
- **Region:** ap-south-1  
- **Worker nodes:** t3a.medium (2 nodes)

# Application Deployment
- **Manifests location:** `k8/manifests/app/`
- **Deployment:** `boardgame-deployment.yaml` (app container from ECR)  
- **Service:** `boardgame-service.yaml` (LoadBalancer for external access)  
- **Accessing the Application:** `kubectl get svc boardgame-service` (retrieve external LoadBalancer endpoint)

# Monitoring with Prometheus & Node Exporter
- **RBAC & ServiceAccount:** grants Prometheus access to resources  
- **Node Exporter:** runs as a DaemonSet, exposes metrics (:9100)  
- **Prometheus ConfigMap:** scrapes Kubernetes nodes (via Node Exporter) and GitHub Runner EC2 (3.110.143.200:9100)  
- **Persistent Storage:** 10Gi PVC (gp2) for TSDB  
- **Prometheus Deployment:** persistent storage + RBAC enabled  
- **Services:** node-exporter → exposes metrics on port 9100, prometheus → exposes Prometheus UI on port 9090  
- **Access:** Local: `kubectl port-forward svc/prometheus -n monitor 9090:9090` → http://localhost:9090;  
- **Metrics Available:** Kubernetes node metrics (CPU, memory, disk, network), GitHub runner EC2 metrics

# Nexus on Kubernetes
- Runs in namespace: nexsus  
- Deployment with PVC for persistent storage  
- Exposed externally via LoadBalancer Service  
- Stores Maven artifacts (snapshots & releases)

# SonarQube on Kubernetes
- Installed via Helm  
- Runs with NodePort service for external access  
- Integrated into CI/CD for static code analysis

# CI/CD Pipeline Overview
- Implemented with GitHub Actions and self-hosted runner  
- **Steps:**  
  1. Checkout Code  
  2. Cache Dependencies: SonarQube cache, Maven dependencies  
  3. Build with Maven: `mvn clean install -DskipTests`  
  4. SonarQube Analysis: uses project key, name, token; runs static code quality analysis  
  5. Deploy Artifacts to Nexus: configure settings.xml with credentials; publish snapshots/releases  
  6. Build Docker Image: creates application container image  
  7. Trivy Security Scan: scans Docker image for vulnerabilities; generates JSON report; uploads report as GitHub artifact; pipeline fails if CRITICAL vulnerabilities found  
  8. Cleanup: deletes Docker image after scan
