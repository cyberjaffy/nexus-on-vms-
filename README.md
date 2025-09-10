# How I Automated Nexus OSS Deployment to AWS VM Using GitHub Actions & Docker

## Overview

This documentation describes the process and technical steps I followed to deploy [Sonatype Nexus Repository OSS](https://www.sonatype.com/products/nexus-repository-oss) on an AWS VM, using Docker and GitHub Actions for full automation. The goal was to create a repeatable, secure, and cloud-native deployment pipeline.

---

## Why This Approach?

- **Automation:** Leveraging GitHub Actions ensures deployments happen on every code push, reducing manual effort.
- **Portability:** Docker containers allow Nexus OSS to run consistently across environments.
- **Cloud integration:** AWS VM hosting provides scalability and reliability for the repository server.

---

## Steps Taken

### 1. **Provisioning the AWS VM**

- I created an Ubuntu-based EC2 instance on AWS.
- Set up a security group to allow inbound traffic on ports 22 (SSH) and 8081 (Nexus UI).

### 2. **Configuring GitHub Secrets**

- Stored VM connection details (`VM_HOST`, `VM_USER`, `VM_KEY`) securely as GitHub Actions secrets.
- This provided safe, automated SSH access from the workflow.

### 3. **Building the GitHub Actions Workflow**

- The workflow is triggered on every push to the `main` branch.
- It checks out the repository, then uses the [appleboy/ssh-action](https://github.com/appleboy/ssh-action) to run remote commands on my VM.

### 4. **Preparing the VM and Deploying Nexus**

- The script ensures Docker is installed and running.
- Stops and removes any old Nexus container, ensuring a clean slate for each deployment.
- Creates a persistent directory `/opt/nexus-data` for Nexus storage, with correct permissions for the container.

### 5. **Running Nexus OSS in Docker**

- Pulls the latest official Nexus OSS Docker image.
- Runs Nexus in a detached container, mapping port 8081 and mounting persistent storage.

### 6. **Configuring the Firewall**

- Opens port 8081 using UFW so the Nexus UI is accessible externally.

### 7. **Initialization and Admin Password Retrieval**

- Waits for Nexus to initialize and generate the default admin password.
- The workflow prints this password in the logs for easy first-time setup.

---

## How to Access Nexus OSS

- **URL:** `http://<your-vm-public-ip>:8081`
- **Admin password:** Check the workflow log output under "Initial Nexus admin password" or SSH to your VM and view the file `/opt/nexus-data/admin.password`.

---

## Lessons Learned & Tips

- **Always check the Nexus Docker image for the latest supported version.**
- **Configure your AWS security group and UFW carefully:** Ensure both SSH and Nexus ports are open before enabling the firewall.
- **Use persistent storage:** This keeps repository data safe even if you redeploy or update the container.
- **Passwordless `sudo` access** for your SSH user will avoid workflow failures.

---

## References

- [Nexus OSS Docker Documentation](https://hub.docker.com/r/sonatype/nexus3)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [appleboy/ssh-action](https://github.com/appleboy/ssh-action)

---

_This approach gives you a fast, reliable, and repeatable way to provision Nexus OSS in the cloud, ready for development or production use._
