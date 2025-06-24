# 1. Install on both Master and Worker Node

# Update package lists
    sudo apt-get update

# Install Docker
    sudo apt install docker.io -y

# Give Docker socket permission (not recommended for production)
    sudo chmod 666 /var/run/docker.sock

# Install dependencies for Kubernetes repo
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Create directory for Kubernetes keyrings
    sudo mkdir -p -m 755 /etc/apt/keyrings

# Add Kubernetes GPG key
     curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes apt repository
     echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update packages
      sudo apt update
