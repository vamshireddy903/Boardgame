# 1. Install Kubeadm
 # follow this 
    https://github.com/Aareez01/kubernetes-v1.30.2-cluster-using-kubeadm
# Download the latest version of kubeadm 

     wget https://github.com/Shopify/kubeaudit/releases/download/v0.22.2/kubeaudit_0.22.2_linux_amd64.tar.gz
 # # Step 2: Extract the archive
    tar -xvzf kubeaudit_0.22.0_linux_amd64.tar.gz

# Step 3: Move the binary to a location in your system's PATH
    sudo mv kubeaudit /usr/local/bin/

# Step 4: Verify installation
    kubeaudit version

# Install Sonarqube

    docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

 # Install nexus

    docker run -d --name Nexus -p 8081:8081 sonatype/nexus3

