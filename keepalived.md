### Installation and Configuration Script:

To set up **Keepalived** with a backup system and priority settings, follow these detailed steps:

### 1. **Install Keepalived on both systems**

Run the following command on both the **master** and **backup** systems to install Keepalived:

```bash
sudo apt update
sudo apt install keepalived
```

### 2. **Configure the Master Node**

Edit the Keepalived configuration file on the **master** system:

```bash
sudo nano /etc/keepalived/keepalived.conf
```

Here is an example configuration for the master node:

```bash
vrrp_instance VI_1 {
    state MASTER               # Set this node as MASTER
    interface eth0              # Network interface (adjust if necessary)
    virtual_router_id 51        # Must be the same on master and backup
    priority 150                # Higher priority for the MASTER
    advert_int 1                # Advertisement interval in seconds
    
    authentication {
        auth_type PASS          # Authentication type
        auth_pass securepass123 # Use a strong password
    }
    
    virtual_ipaddress {
        192.168.188.2           # The virtual IP that will float between systems
    }
}
```

- `state MASTER`: Defines the system as the master node.
- `priority 100`: The master node has a higher priority. The backup node will have a lower priority.
- `virtual_router_id`: Must be the same across both master and backup nodes.

### 3. **Configure the Backup Node**

Now, on the **backup** system, create or edit the Keepalived configuration file:

```bash
sudo nano /etc/keepalived/keepalived.conf
```

Here is an example configuration for the backup node:

```bash
vrrp_instance VI_1 {
    state BACKUP                # Set this node as BACKUP
    interface eth0              # Network interface
    virtual_router_id 51        # Same as the master
    priority 90                 # Lower priority than the master
    advert_int 1                # Same advertisement interval as the master
    
    authentication {
        auth_type PASS          # Same authentication method
        auth_pass securepass123 # Same password as the master
    }
    
    virtual_ipaddress {
        192.168.188.2           # The same virtual IP
    }
}
```

- `state BACKUP`: Defines the system as the backup node.
- `priority 90`: Lower priority so that the backup only takes over when the master fails.
- The other settings, such as `virtual_router_id` and `auth_pass`, must match the master configuration.

### 4. **Start and Enable Keepalived**

After configuring Keepalived on both systems, enable and start the service on both:

```bash
sudo systemctl enable keepalived
sudo systemctl start keepalived
```

### 5. **Testing Failover**

To test the failover, you can stop the Keepalived service on the master node:

```bash
sudo systemctl stop keepalived
```

On the backup node, check if it takes over the virtual IP:

```bash
ip a
```

You should see the virtual IP `192.168.188.2` assigned to the backup node’s interface.

### Additional Details:

- **Priority**: In a Keepalived setup, the node with the highest priority becomes the master. If the master fails, the backup with the next highest priority takes over.
- **Failback**: When the master node recovers, it will automatically reclaim the virtual IP unless you configure `nopreempt`. 

### 6. **Monitoring**

To monitor the status of the Keepalived service, use:

```bash
sudo systemctl status keepalived
```

You can also check logs for any issues:

```bash
sudo tail -f /var/log/syslog
```

By following these steps, you will have a fully functioning Keepalived setup with master/backup failover and a shared virtual IP (`192.168.188.2`).




























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
