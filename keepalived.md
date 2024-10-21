
### Installation and Configuration Script:

```bash
#!/bin/bash

# Step 1: Install Keepalived
echo "Installing Keepalived..."
sudo apt-get update -y
sudo apt-get install -y keepalived

# For RHEL/CentOS-based systems:
# sudo yum install -y keepalived

# Step 2: Backup existing Keepalived configuration
echo "Backing up existing configuration..."
sudo cp /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak

# Step 3: Define new Keepalived configuration
echo "Configuring Keepalived..."

# Example configuration file
cat <<EOT | sudo tee /etc/keepalived/keepalived.conf
vrrp_instance VI_1 {
    state MASTER        # MASTER for main server, BACKUP for backup server
    interface eth0      # The network interface for VRRP communication (e.g., eth0)
    virtual_router_id 51
    priority 100        # Higher priority for MASTER (lower for BACKUP)
    advert_int 1        # Advertisement interval (1 second)
    
    authentication {
        auth_type PASS  # Simple password-based authentication
        auth_pass securepass123  # Change this to a strong password
    }
    
    virtual_ipaddress {
        192.168.188.2  # Virtual IP address (the one to be moved between servers)
    }
}

# You can add more VRRP instances for different services or failover setups.
EOT

# Step 4: Enable and start Keepalived service
echo "Enabling and starting Keepalived service..."
sudo systemctl enable keepalived
sudo systemctl start keepalived

# Step 5: Verify Keepalived status
echo "Checking Keepalived status..."
sudo systemctl status keepalived

# Print a final message
echo "Keepalived installation and configuration completed."
```

### Explanation:

1. **Install Keepalived**: This script installs the Keepalived package, a service that handles failover and load balancing via VRRP.
2. **Backup Configuration**: It backs up the existing configuration for safety.
3. **Configure Keepalived**: It sets up a basic Keepalived configuration, which includes:
   - **State**: `MASTER` for the main server, `BACKUP` for others.
   - **Priority**: The higher the priority, the more likely this node will be the master.
   - **Virtual IP**: The floating IP address that will be shared between the master and backup nodes.
   - **Authentication**: A simple password-based authentication for VRRP.
4. **Enable and Start**: The script enables and starts the Keepalived service to run at boot.
5. **Status Check**: Finally, it checks the status of the service.

### Additional Notes:
- On a backup server, the configuration will be similar, but you should change the state to `BACKUP` and reduce the priority to a lower number (e.g., 90).
- Be sure to update the network interface (`eth0`) to match your system's network configuration.
