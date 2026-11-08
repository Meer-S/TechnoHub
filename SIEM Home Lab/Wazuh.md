Wazuh SIEM Home Lab - Multi-Network Ubuntu, Apache HTTPS & Agent Integration
            Project Overview
            
       This project documents the design, implementation, and troubleshooting of a cybersecurity home lab consisting of:
            
            Wazuh Manager
            Ubuntu 24.04/26.04 Endpoints
            Metasploitable 2 Vulnerable VM
            VMware Networking
            Apache2 Web Server
            HTTPS using Self-Signed Certificates
            Wazuh Agent Enrollment
            Multi-NIC Network Configuration
            
      The goal was to simulate a real-world SOC environment where endpoints communicate with the Wazuh Manager while exposing secure web 
      applications protected by TLS.
            
                                           Lab Architecture
                                              Internet
                                                  │
                                                  │
                                      Home Router (192.168.1.1)
                                                  │
                      ┌───────────────────────────┼───────────────────────────┐
                      │                           │                           │
                      │                           │                           │
                Physical Host              Ubuntu Web Server           Wazuh Manager
                192.168.1.x                192.168.1.50               10.10.10.134
                                           10.10.10.131
                                                 │
                                  VMware Host-Only Network
                                           10.10.10.0/24
                                                 │
                             ┌───────────────────┼───────────────────┐
                             │                   │                   │
                             │                   │                   │
                     Ubuntu Agent         Metasploitable 2      Future Agents
                     10.10.10.131         10.10.10.128          Windows/Linux
        
        Technologies Used
        Component	TechnologySIEM	Wazuh 4.14
        Operating System	Ubuntu 24.04 / 26.04
        Web Server	Apache2
        Virtualization	VMware Workstation
        PKI	OpenSSL
        Secure Transport	TLS 1.2 / HTTPS
        Vulnerable Target	Metasploitable 2
        Networking	Netplan, NetworkManager
        Monitoring	Wazuh Agents
        Objectives
        Deploy a Wazuh security monitoring environment.
        Configure endpoint agents.
        Build a segmented VMware network.
        Enable HTTPS using self-signed certificates.
        Validate secure communication.
        Troubleshoot real-world configuration issues.
        Network Configuration
        Interface Discovery
        
        Detected interfaces:
        
        ip a
        
        
        Output:
        
        ens33 -> 10.10.10.131/24
        ens37 -> 192.168.1.50/24
        
        ens33
        
        Purpose:
        
        Host-only / Internal Lab Network
        
        
        IP:
        
        10.10.10.131
        
        ens37
        
        Purpose:
        
        Bridged Network
        
        
        IP:
        
        192.168.1.50
        
        
        Gateway:
        
        192.168.1.1
        
        Netplan Configuration
        
        File:
        
        /etc/netplan/01-network-manager-all.yaml
        
        
        Example:
        
        network:
          version: 2
          renderer: NetworkManager
        
          ethernets:
            ens33:
              dhcp4: true
        
            ens37:
              dhcp4: false
              addresses:
                - 192.168.1.50/24
        
              routes:
                - to: default
                  via: 192.168.1.1
        
              nameservers:
                addresses:
                  - 8.8.8.8
                  - 1.1.1.1
        
        
        Apply:
        
        sudo netplan generate
        sudo netplan apply
        
        Apache HTTPS Configuration
        Enable SSL
        sudo a2enmod ssl
        sudo a2ensite default-ssl
        
        
        Restart:
        
        sudo systemctl restart apache2
        
        Generate Self-Signed Certificate
        sudo openssl req -x509 -nodes -days 365 \
        -newkey rsa:4096 \
        -keyout /etc/ssl/private/apache-selfsigned.key \
        -out /etc/ssl/certs/apache-selfsigned.crt
        
        Apache SSL Configuration
        
        File:
        
        /etc/apache2/sites-available/default-ssl.conf
        
        
        Configuration:
        
        SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
        SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
        
        Subject Alternative Names (SAN)
        
        Certificate configured with:
        
        DNS: ubuntu2604.lab.local
        IP : 192.168.1.13
        IP : 10.10.10.131
        
        
        Validation:
        
        openssl x509 -in /etc/ssl/certs/apache-selfsigned.crt \
        -text -noout
        
        
        Output:
        
        X509v3 Subject Alternative Name:
        DNS:ubuntu2604.lab.local
        IP Address:192.168.1.13
        IP Address:10.10.10.131
        
        Wazuh Agent Enrollment
        Create Agent
        sudo /var/ossec/bin/manage_agents
        
        
        Select:
        
        A - Add Agent
        
        Extract Agent Key
        sudo /var/ossec/bin/manage_agents
        
        
        Select:
        
        E - Extract Key
        
        
        Example:
        
        MDAyIHVidW50dTI2MDQgYW55I...
        
        Import Agent Key
        
        Agent Host:
        
        sudo /var/ossec/bin/manage_agents
        
        
        Select:
        
        I - Import Key
        
        
        Paste extracted key.
        
        Start Agent
        sudo systemctl restart wazuh-agent
        sudo systemctl enable wazuh-agent
        
        Validation Checks
        Apache
        sudo apache2ctl configtest
        
        
        Expected:
        
        Syntax OK
        
        HTTPS Validation
        curl -k https://localhost
        
        
        Expected:
        
        Apache Default Page
        
        Agent Validation
        sudo systemctl status wazuh-agent
        
        
        Expected:
        
        active (running)
        
        Troubleshooting Log
        Issue 1
        Error
        Connection refused
        
        Root Cause
        
        SSH service not running.
        
        Resolution
        sudo systemctl start ssh
        sudo systemctl enable ssh
        
        Issue 2
        Error
        No matching host key type found.
        Their offer: ssh-rsa, ssh-dss
        
        Root Cause
        
        Legacy Metasploitable SSH implementation.
        
        Resolution
        ssh -o HostKeyAlgorithms=+ssh-rsa msfadmin@10.10.10.128
        
        Issue 3
        Error
        SSL handshake failure
        
        Root Cause
        
        Metasploitable OpenSSL version too old.
        
        Resolution
        
        Package downloaded externally and transferred manually.
        
        Issue 4
        Error
        AH00526: SSLCertificateFile does not exist
        
        Root Cause
        
        Typographical error:
        
        apache-selffigned.crt
        
        Resolution
        
        Corrected:
        
        apache-selfsigned.crt
        
        Issue 5
        Error
        Permissions for netplan YAML are too open
        
        Resolution
        sudo chmod 600 /etc/netplan/*.yaml
        
        Security Considerations
        Risks
        Self-Signed Certificate
        
        Risk:
        
        Browser trust warnings
        
        
        Mitigation:
        
        Install root certificate into Trusted Root Store
        
        Legacy Metasploitable Services
        
        Risk:
        
        Known vulnerable services
        
        
        Mitigation:
        
        Isolated lab network only
        
        Lessons Learned
        VMware networking requires clear separation of NAT, Bridged, and Host-Only modes.
        Wazuh agents require network reachability before enrollment.
        SAN certificates are mandatory for modern browsers.
        Apache SSL failures are often caused by certificate path issues.
        Netplan configurations should use actual interface names discovered through ip a.
        Multi-NIC Linux systems require careful routing and gateway planning.
        Legacy systems frequently fail TLS negotiations with modern repositories.
        Evidence
        Apache HTTPS
        https://192.168.1.13
        
        
        Loaded successfully using self-signed certificate.
        
        Certificate Validation
        DNS:ubuntu2604.lab.local
        IP:192.168.1.13
        IP:10.10.10.131
        
        
        Successfully embedded in SAN extension.
        
        Wazuh Agent
        
        Agent enrolled and communicating with manager.
        
        Future Enhancements
        Integrate pfSense firewall.
        Enable Suricata IDS/IPS.
        Deploy additional Linux and Windows agents.
        Configure Wazuh Active Response.
        Integrate VirusTotal.
        Enable vulnerability detection.
        Build attack simulation scenarios using Metasploitable.
        Author
        
        Meer Saifulla
         Cybersecurity Consultant | Wazuh SIEM | DLP | SASE | AI Security Automation | VMware Labs
        
        This README is already GitHub-ready and follows a professional project documentation structure suitable for a cybersecurity portfolio repository.
