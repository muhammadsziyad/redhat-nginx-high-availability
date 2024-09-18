
# High Availability NGINX Setup on Red Hat Enterprise Linux (RHEL)

## Objective

The objective of this project is to set up a high availability (HA) NGINX environment on Red Hat 
Enterprise Linux (RHEL) to ensure continuous availability and reliability of web services. The setup will 
include a primary and secondary NGINX server with load balancing and failover capabilities.

```mermaid
sequenceDiagram
    participant Client
    participant VIP as Virtual IP (VIP)
    participant NGINX1 as NGINX Server 1 (Primary)
    participant NGINX2 as NGINX Server 2 (Secondary)
    participant Backend as Backend Server

    Client->>VIP: Send Request
    VIP->>NGINX1: Forward Request (if Primary is active)
    NGINX1->>NGINX1: Check Load Balancing Rules
    NGINX1->>Backend: Forward Request
    Backend->>Backend: Process Request
    Backend->>NGINX1: Send Response
    NGINX1->>VIP: Forward Response
    VIP->>Client: Send Response

    Note over VIP: Optional Failover Check
    VIP->>NGINX1: Check Server Status
    alt Primary Server Active
        NGINX1->>VIP: Continue Handling Requests
    else Primary Server Down
        VIP->>NGINX2: Failover to Secondary Server
        NGINX2->>NGINX2: Take Over VIP
        NGINX2->>VIP: Continue Handling Requests
    end
   ```

## Project Overview

1.  **Planning and Preparation**
    
    -   Define the high availability requirements.
    -   Prepare the RHEL environment.
    -   Install NGINX on multiple servers.
2.  **Configure NGINX for Load Balancing**
    
    -   Set up NGINX as a load balancer.
    -   Configure backend servers.
    -   Test load balancing configuration.
3.  **Configure High Availability**
    
    -   Set up a Virtual IP (VIP) with failover using Keepalived.
    -   Configure Keepalived for high availability.
    -   Test failover functionality.
4.  **Testing and Validation**
    
    -   Verify load balancing and failover.
    -   Ensure proper failover and recovery mechanisms.
5.  **Documentation and Conclusion**
    
    -   Document the setup process.
    -   Evaluate benefits, challenges, and performance.

## Step-by-Step Configuration

### 1. Planning and Preparation

-   **Install RHEL**: Ensure RHEL is installed and updated on both NGINX servers.
-   **Update System**:

    `sudo yum update -y` 
    

### 2. Install NGINX on Both Servers

#### On Primary and Secondary NGINX Servers

-   **Install NGINX**:
  
    `sudo yum install epel-release -y
    sudo yum install nginx -y` 
    
-   **Start and Enable NGINX Service**:
    

    
    `sudo systemctl start nginx
    sudo systemctl enable nginx` 
    

### 3. Configure NGINX for Load Balancing

#### On Both NGINX Servers

-   **Edit NGINX Configuration**:
    -   Create or modify the `/etc/nginx/nginx.conf` file or create a new configuration file in 
`/etc/nginx/conf.d/` (e.g., `load-balancer.conf`):
        

        
        ```
        upstream backend {
            server backend1.example.com;
            server backend2.example.com;
        }
        
        server {
            listen 80;
            server_name yourdomain.com;
        
            location / {
                proxy_pass http://backend;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
            }
        }
        ```
        
-   **Reload NGINX Configuration**:
    
    `sudo systemctl reload nginx` 
    

### 4. Configure High Availability with Keepalived

#### On Both NGINX Servers

-   **Install Keepalived**:

    
    `sudo yum install keepalived -y` 
    
-   **Configure Keepalived**:
    
    -   Edit the `/etc/keepalived/keepalived.conf` file on both servers. Here’s an example configuration 
for the primary server:

        
        ```
        vrrp_instance VI_1 {
            state MASTER
            interface eth0
            virtual_router_id 51
            priority 101
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 1234
            }
            virtual_ipaddress {
                192.168.1.100
            }
        }
        ``` 
        
        -   On the secondary server, use `state BACKUP` and a lower `priority` (e.g., 100).
-   **Start and Enable Keepalived**:

    
    `sudo systemctl start keepalived
    sudo systemctl enable keepalived` 
    

### 5. Testing and Validation

-   **Verify Load Balancing**:
    -   Access `http://yourdomain.com` and ensure traffic is distributed across backend servers.
-   **Verify Failover**:
    -   Simulate a failure of the primary server and ensure that the secondary server takes over the 
Virtual IP (VIP).

### 6. Documentation and Conclusion

-   **Document Configuration**: Record all installation and configuration steps, including NGINX and 
Keepalived settings.
-   **Evaluate Benefits**:
    -   **High Availability**: Ensures continuous operation even if one server fails.
    -   **Load Balancing**: Distributes incoming requests across multiple backend servers.
-   **Challenges**:
    -   **Configuration Complexity**: Properly configuring NGINX and Keepalived for HA.
    -   **Testing Failover**: Ensuring seamless failover and recovery.
