# GitHub CI/CD Pipeline for Python Web Application

## Overview
This CI/CD pipeline automates the deployment of a Python web application to a remote server using SSH and rsync. The pipeline ensures seamless deployment by synchronizing files, managing virtual environments, and restarting the application service.

## Pipeline Configuration

### **Stages**
The pipeline consists of the following stage:
- **Deploy**: Transfers files to the remote server, sets up the virtual environment, and restarts the application.

### **Environment Variables**
The following environment variables should be configured in GitHub Actions Secrets:
- `SSH_USER` - The username for SSH access.
- `SSH_HOST` - The remote server hostname or IP.
- `SSH_KEY` - The private SSH key for authentication.
- `APP_DIR` - The directory where the application will be deployed.

## **Deployment Workflow**

### **1. Setup SSH Authentication**
```bash
eval $(ssh-agent -s)
echo "$SSH_KEY" | tr -d '\r' | ssh-add -
mkdir -p ~/.ssh
echo -e "Host *\n\tStrictHostKeyChecking no\n" > ~/.ssh/config
```
- Starts the SSH agent and adds the SSH private key.
- Disables strict host key checking to avoid SSH confirmation prompts.

### **2. Create Deployment Directory**
```bash
ssh $SSH_USER@$SSH_HOST "mkdir -p $APP_DIR"
```
- Ensures that the deployment directory exists on the remote server.

### **3. Sync Code to Remote Server**
```bash
rsync -avz --exclude='.git' --exclude='venv' ./ $SSH_USER@$SSH_HOST:$APP_DIR
```
- Uses `rsync` to transfer files to the remote server.
- Excludes unnecessary directories such as `.git` and `venv`.

### **4. Create and Upload Deployment Script**
```bash
echo -e "#!/bin/bash\n
cd $APP_DIR\nif [ ! -d 'venv' ]; then\n  python3 -m venv venv\nfi\nsource venv/bin/activate\nsudo systemctl restart gyansutra\n" > deploy.sh
rsync -avz deploy.sh $SSH_USER@$SSH_HOST:$APP_DIR/deploy.sh
```
- Creates a deployment script (`deploy.sh`) on the local machine.
- Ensures a Python virtual environment exists.
- Activates the virtual environment.
- Restarts the `gyansutra` service.
- Uploads `deploy.sh` to the remote server.

### **5. Set Permissions and Execute Deployment Script**
```bash
ssh $SSH_USER@$SSH_HOST "chmod +x $APP_DIR/deploy.sh"
ssh $SSH_USER@$SSH_HOST "$APP_DIR/deploy.sh"
```
- Grants execution permissions to `deploy.sh`.
- Runs the deployment script on the remote server.

## **Deployment Conditions**
```yaml
only:
  - main  # Deploys only from the main branch
```
- Ensures that deployments only occur when changes are pushed to the `main` branch.

## **Prerequisites**
- The remote server must allow SSH access using GitHub SSH keys.
- The `rsync` package must be installed on both the local and remote machines.
- The remote server must have Python installed.
- The application should be managed by `systemd` (e.g., `gyansutra` service).
- Configure GitHub Actions Secrets with the required environment variables.

## **Summary**
- Automates deployment of the Python web application.
- Uses SSH and `rsync` for file synchronization.
- Manages virtual environments dynamically.
- Ensures safe deployment through GitHub Actions Secrets.
- Restarts the application service after deployment.

This pipeline ensures a smooth deployment process for Python web applications. 🚀

