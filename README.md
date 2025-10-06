# Euro-mediterranean Centre on Climate Change Foundation - CMCC
## Advanced Digital Innovation Centre - ADIC
## Job Call 12757 - 2 Cloud System Administrators Junior and/or Senior
### Name: Ali Erdem Kökcik
### First Interview: 23/09/2025
### Invitation date: XXXXXXXXX
### Due Date: XXXXXXXXX
### Exercise: Deployment of a JupyterHub instance
### Objective:
#### Deploy a [JupyterHub](https://jupyter.org/) instance using a [minikube](https://minikube.sigs.k8s.io/docs/) environment (or equivalent). The deployment must have two PersistentVolumeClaims (PVCs) of different sizes: 
- one PVC should be configured as read-only and used for hosting input data
- the other should be read-write for storing “jupyter notebooks”, results and any material associated with each user's workspace.

### Requirements:
#### JupyterLab Deployment:
- Package a JupyterHub instance in a Kubernetes Deployment.
- Authentication: Provide basic authentication based on Dummy Authenticator
- Ingress/Service Exposure: add an Ingress or Service configuration to expose JupyterHub externally.

#### PersistentVolumeClaims:
##### Data Volume:
- Create a PVC intended for input data processing.
- Configure it with a smaller size (e.g., 5Gi) and mount it as read-only.
##### Notebook Volume:
- Create a PVC intended for notebooks and output storage.
- Configure it with a larger size (e.g., 20Gi) and allow read-write access.

#### Documentation:
##### Provide a clear and concise README explaining:
- How to install, upgrade, and uninstall the deployment.
- How to customize the PVC sizes, mount paths, and any other configurable parameters.

### Optional Enhancements:
- Readiness/Liveness: Optionally add readiness/liveness checks in order to enhance the reliability and stability of the application.
- Security: Consider implementing a securityContext in the pod spec (e.g., running as non-root).
- Enhance the authentication method, by taking advantage of external identity provider or OAuth service (e.g. Github/Google Login)

### Evaluation Criteria:
#### Functionality:
- A JupyterHub instance is succeffully deployed on top of a Kubernetes cluster.
- Both PVCs are created with the specified configurations: one read-only for input data and one read-write for writing notebooks.
#### Code Quality:
- The deployment follows the best practices with clear templating, proper structure, and appropriate use of labels and annotations.
#### Documentation:
- The README provides clear, concise instructions on usage and customization.

### Additional Suggestions:
#### Resource Management:
- Consider adding resource requests and limits to the JupyterHub container for a better cluster resource management.
#### Custom Environment Variables:
- Allow passing custom environment variables (e.g., for configuring JupyterLab settings) via ConfigMaps

#### This exercise is designed to test the candidates' practical skills with Kubernetes, Docker and stateful applications in a cloud environment. It also evaluates their ability to write clear documentation and manage configurable parameters.
