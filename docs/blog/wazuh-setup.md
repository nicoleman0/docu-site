# Building A Home SIEM with Wazuh

As part of developing my cybersecurity skills, I decided to set up a **Wazuh** a server to monitor my home network.

Wazuh is an open-source platform (SIEM) that provides log analysis, intrusion detection, vulnerability monitoring, and more. It's a great tool for blue teamers and aspiring SOC analysts, as it gives you some hands on practice with monitoring a network for security incidents.

---

## Why Wazuh

I chose Wazuh because:

- It’s completely free and open source.
- It integrates easily with multiple endpoints.
- It provides detailed dashboards for monitoring system security.
- It’s used by professionals for SIEM and endpoint detection & response (EDR).

My goal was to gain real-world, hands-on experience with log collection, alerting, and analysis — all within a **home lab** environment.

---

## System setup

### **Environment**

- **Host Machine:** Ubuntu 22.04 (MSI Stealth 15M)
- **Endpoints:** Ubuntu laptop, MacBook Air (will add the rest of my family's later)
- **Network:** Home Wi-Fi router with local IP addressing (192.168.x.x)
- **Installation Type:** Wazuh single-node server with web dashboard (VirtualBox)

---

## Step 1: Install the Wazuh Server

In my opinion, the easiest way to get started with Wazuh is by setting up a Virtual Machine server, using the [official Wazuh .ova file.](https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html) Then you can import it into your hypervisor of choice - I used VirtualBox.

The preconfigured image comes with the Wazuh Manager, Filebeat, Elasticsearch, and the Wazuh Dashboard already set up, which saves a ton of time compared to installing each component manually. Seriously.

Once imported, make sure your VM settings are correct:

- **Memory (RAM)**: Allocate at least 4 GB — more if you plan to monitor multiple endpoints.
- **CPU**: 2 cores minimum is recommended for smoother performance.
- **Network Adapter**: Use Bridged Adapter mode so the server appears on your local network and can communicate with your agents directly.

After starting the VM, Wazuh will automatically initialize its services. When it’s finished, you can check the dashboard by accessing it online via the VM's IP address.

```md
https://<your-wazuh-server-ip>
```

Then you can login with the default credentials, which should be *admin* and *admin*.

## Step 2: Add your endpoint agents

With your Wazuh server up and running, the next step is to connect your endpoints (the systems you want to monitor). In my case, I started with my Ubuntu laptop and MacBook Air. I plan on adding more, with the permission of my relatives.

Each endpoint needs a Wazuh agent installed. The agent collects logs, monitors system activity, and sends that data to your Wazuh server for analysis. It acts like a central command, that is fed data from its various sources.

### Get the agent installation command

From your Wazuh Dashboard:

- Go to “Agents” → “Deploy new agent.”
- Select your endpoint’s operating system and CPU architecture.
- Include the IP of your Wazuh server.
- Wazuh will then automatically generate an installation command for you.

### Run the command on the endpoint

On your endpoint machine, open a terminal and run the command provided by Wazuh.
It will look something like this:

```bash
curl -so wazuh-agent.sh https://packages.wazuh.com/4.x/wazuh-agent.sh && \
sudo bash wazuh-agent.sh --register-server <WAZUH_SERVER_IP> --agent-name <ENDPOINT_NAME>
```

Once the installation finishes, start and enable the Wazuh agent:

```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### Confirm the agent is active

Head back to the Wazuh Dashboard → Agents tab.

You should now see your new endpoint listed with a status of “Active.”

- If it’s showing as pending or disconnected, check your network configuration and ensure your server and agent are on the same subnet.

## Step 3: Verify Data Flow

Once your agent is active, logs should start appearing almost immediately in the dashboard.

Try generating some activity on your endpoint — running commands, updating packages, or even logging in and out — to watch Wazuh capture the corresponding events. I was personally surprised to see it capture my MacBook's screen being locked. 

Wazuh will also reccomend fixes for various vulnerabilities. Don't be too nervous if you see a bunch of them at first, most of the time these vulnerabilities are nothing to be that worried about. They can be fixed simply, if you follow the directions.

## Next Steps

Getting Wazuh up and running is only the beginning for me. Now that my server is collecting logs and monitoring activity across multiple endpoints, I want to take things further and make my home SIEM more capable and realistic.

### Network Traffic Analysis

My main goal right now is to integrate network traffic monitoring. By default, Wazuh focuses on endpoint activity — logins, commands, system alerts, and vulnerabilities. The next step is to add network-level visibility so I can see which devices are communicating, what services they’re using, and whether anything suspicious is happening on the network.

I’m currently exploring ways to combine Wazuh with Zeek or Suricata, or to export logs directly from my router/firewall (although not sure how leninent BT is with this sort of stuff) into Wazuh for deeper insight into network behavior.

### Nessus Compatibility

While Wazuh’s built-in vulnerability detection is powerful, I want to pair it with a dedicated scanner. My MacBook doubles as a Nessus server, so I plan to feed scan results into Wazuh to track vulnerabilities and remediation over time.

### Log Retention

As data grows, storage management becomes increasingly important. I plan to experiment with remote log storage and log rotation strategies to keep performance stable while still retaining useful historical data for analysis.

## Reflecting

Setting up Wazuh has been a very rewarding experience, and given me hands-on experience with real-world SIEM concepts such as log collection and analysis to alerting and vulnerability monitoring.

There’s still a lot to learn, but that is the point! As I continue improving my setup with network traffic monitoring, Nessus integration, and better data retention, I’m getting closer to understanding what it’s like to operate in a professional SOC environment.

If you’re interested in cybersecurity or want to practice blue team skills, I highly recommend trying out Wazuh yourself. It's not that difficult (if setting up as a dedicated VM) and you’ll learn a ton about how systems communicate, how attacks are detected, and how data becomes visibility.
