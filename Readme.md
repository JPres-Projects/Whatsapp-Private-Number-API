# WhatsApp Private Number API

A basic self-hosted REST API that connects to your personal WhatsApp account, allowing you to send and receive messages programmatically from your own phone number. This enables you to integrate WhatsApp messaging into your applications, automation systems, or scripts without relying on third-party services.

This API is ideal for localhost use cases, as other solutions offer more extensive data sending and retrieval capabilities.

## 🎯 Purpose & Use Cases

**Turn your private WhatsApp number into a programmable API:**
- **N8N Automation**: Monitor incoming WhatsApp messages and send automated responses from your private number
- **Workflow Integration**: Connect email notifications, CRM updates, or any N8N workflow to send WhatsApp messages  
- **Server Monitoring**: Get alerts and status updates directly on your personal WhatsApp
- **Custom Notifications**: Send automated messages from your own number for home automation, reminders, or alerts
- **Script Integration**: Any application that can make HTTP requests can send WhatsApp messages from your private number

**Session Persistence**: The connection remains active as long as your phone number is actively used. According to Reddit users, sessions stay active for up to 14 days if the phone is used within that timeframe. After 14 days of phone inactivity, a new QR code scan is required.

## ⚠️ Important Warning

This project uses an unofficial method to communicate with WhatsApp. **This is against the WhatsApp Terms of Service.** There is a real risk that your phone number could be temporarily or permanently banned by WhatsApp.

**Use at your own risk. Recommended to use a secondary, non-critical phone number for development.**

## ✨ Features

- **REST API Interface**: Simple HTTP endpoints for sending messages
- **Media Support**: Send images, audio, videos, and documents
- **Self-Hosted**: Runs entirely on your own infrastructure
- **Session Persistence**: After QR code scan, automatically reconnects
- **24/7 Operation**: Can run as a system service
- **External Access**: Cloudflare Tunnel integration for worldwide access
- **Multi-Device Support**: Works alongside WhatsApp Web and mobile app

## 🚀 Quick Start

### Prerequisites

- Ubuntu 20.04+ server (Oracle Cloud, DigitalOcean, AWS, etc.)
- Go 1.21+ installed
- Internet connection
- WhatsApp account

### Installation

```bash
# Clone the repository
git clone https://github.com/JPres-Projects/Basic-Whatsapp-Private-API.git
cd Whatsapp-Private-Number-API/whatsapp-bridge

# Install dependencies
sudo apt update
sudo apt install golang-go build-essential -y

# Build the application
export CGO_ENABLED=1
go mod tidy
go build -o whatsapp-api main.go

# First run (QR code authentication)
./whatsapp-api
```

### Initial Setup

1. **QR Code Authentication**: On first run, scan the QR code with your WhatsApp mobile app:
   - Open WhatsApp → Settings → Linked Devices → Link a Device
   - Scan the QR code displayed in terminal

2. **Verify Connection**: After successful scan, you'll see:
   ```
   ✓ Connected to WhatsApp! 
   Starting REST API server on :8080...
   ```

3. **Test the API**:
   ```bash
   # In a new terminal
   curl -X POST http://localhost:8080/api/send \
     -H "Content-Type: application/json" \
     -d '{"recipient":"1234567890","message":"Hello from API!"}'
   ```

## 🔗 API Endpoints

### Send Message
```http
POST /api/send
Content-Type: application/json

{
  "recipient": "1234567890",      // Phone number in international format
  "message": "Hello World!",      // Text message
  "media_path": "/path/to/file"   // Optional: local file path
}
```

### Download Media
```http
POST /api/download
Content-Type: application/json

{
  "message_id": "msg_id",
  "chat_jid": "1234567890@s.whatsapp.net"
}
```

### Examples

**Text Message:**
```bash
curl -X POST http://localhost:8080/api/send \
  -H "Content-Type: application/json" \
  -d '{"recipient":"1234567890","message":"Hello from API!"}'
```

**Media Message:**
```bash
curl -X POST http://localhost:8080/api/send \
  -H "Content-Type: application/json" \
  -d '{"recipient":"1234567890","message":"Check this image!","media_path":"/home/user/image.jpg"}'
```

**Group Message:**
```bash
curl -X POST http://localhost:8080/api/send \
  -H "Content-Type: application/json" \
  -d '{"recipient":"1234567890@g.us","message":"Hello group!"}'
```

## 🏗️ Oracle Cloud Server Setup

### Firewall Configuration

```bash
# Update system
sudo apt-get update
sudo apt-get install iptables-persistent -y

# Open required ports
sudo iptables -I INPUT 6 -p tcp --dport 8080 -j ACCEPT
sudo iptables -I INPUT 6 -p tcp --dport 80 -j ACCEPT  
sudo iptables -I INPUT 6 -p tcp --dport 443 -j ACCEPT

# Save rules permanently
sudo netfilter-persistent save
```

### Security Groups (Oracle Cloud Console)
Add these **Ingress Rules**:
- **Port 80**: Source 0.0.0.0/0 (HTTP)
- **Port 443**: Source 0.0.0.0/0 (HTTPS)
- **Port 8080**: Source 0.0.0.0/0 (API - optional if using Cloudflare)

📖 **For detailed Oracle Cloud setup**: [Oracle Cloud General Setup Guide](https://github.com/JPresting/3---Setup-Guides/tree/main/Oracle-Cloud)

## ☁️ Cloudflare Tunnel Setup (External Access)

Enable worldwide access to your WhatsApp API through Cloudflare Tunnel.

### Quick Setup

**Windows (Local Machine):**
```powershell
# Download cloudflared
Invoke-WebRequest https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe -OutFile cloudflared.exe

# Authenticate
.\cloudflared.exe tunnel login

# Create tunnel
.\cloudflared.exe tunnel create whatsapp-api

# Set DNS
.\cloudflared.exe tunnel route dns whatsapp-api api.yourdomain.com
```

**Ubuntu Server:**
```bash
# Install cloudflared
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb
sudo dpkg -i cloudflared-linux-arm64.deb

# Create config
sudo nano /etc/cloudflared/config.yml
```

**Config File:**
```yaml
tunnel: whatsapp-api
credentials-file: /home/ubuntu/.cloudflared/TUNNEL-ID.json

ingress:
  - hostname: api.yourdomain.com
    service: http://127.0.0.1:8080
  - service: http_status:404
```

```bash
# Install and start service
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

📖 **For detailed Cloudflare setup**: [Cloudflare Tunnel Setup Guide](https://github.com/JPresting/3---Setup-Guides/tree/main/Cloudflare/Cloudflare-Tunnel-Setup)

## 🔄 Permanent Service Setup

Run WhatsApp API as a system service for 24/7 operation.

### Create Service

```bash
# Create service file
sudo nano /etc/systemd/system/whatsapp-api.service
```

**Service Configuration:**
```ini
[Unit]
Description=WhatsApp API Service
After=network.target

[Service]
Type=simple
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/Whatsapp-Private-Number-API/whatsapp-bridge
ExecStart=/home/ubuntu/Whatsapp-Private-Number-API/whatsapp-bridge/whatsapp-api
Restart=always
RestartSec=10
Environment=CGO_ENABLED=1

[Install]
WantedBy=multi-user.target
```

### Enable Service

```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable auto-start
sudo systemctl enable whatsapp-api

# Start service
sudo systemctl start whatsapp-api

# Check status
sudo systemctl status whatsapp-api

# View logs
sudo journalctl -u whatsapp-api -f
```

### Service Management

```bash
# Stop service
sudo systemctl stop whatsapp-api

# Restart service
sudo systemctl restart whatsapp-api

# Disable auto-start
sudo systemctl disable whatsapp-api

# Remove service completely
sudo rm /etc/systemd/system/whatsapp-api.service
sudo systemctl daemon-reload
```

## 🌐 External API Usage

After Cloudflare Tunnel setup, access your API from anywhere:

```bash
# From any internet-connected device
curl -X POST https://api.yourdomain.com/api/send \
  -H "Content-Type: application/json" \
  -d '{"recipient":"1234567890","message":"Message from anywhere!"}'
```

### N8N Integration

**HTTP Request Node Configuration:**
```json
{
  "method": "POST",
  "url": "https://api.yourdomain.com/api/send",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {
    "recipient": "{{$json.phone_number}}",
    "message": "New email from {{$json.sender}}: {{$json.subject}}"
  }
}
```

## 🔧 Troubleshooting

### Common Issues

**QR Code Not Appearing:**
```bash
# Clear session and restart
rm -rf store/
./whatsapp-api
```

**API Not Responding:**
```bash
# Check if service is running
sudo systemctl status whatsapp-api

# Check port binding
sudo netstat -tulpn | grep 8080
```

**Connection Lost:**
```bash
# Restart service
sudo systemctl restart whatsapp-api

# Check logs
sudo journalctl -u whatsapp-api -f
```

**Cloudflare Tunnel Issues:**
```bash
# Check tunnel status
sudo systemctl status cloudflared

# Restart tunnel
sudo systemctl restart cloudflared
```

### Session Management

- **Session Duration**: WhatsApp sessions last up to 14 days if your phone number remains actively used
- **Re-authentication**: Required after 14 days of phone inactivity (not API inactivity)
- **Session Storage**: All credentials stored in `store/` directory
- **Backup**: Regular backup of `store/` directory recommended

### Security Notes

- Keep `store/` directory private and secure
- Use strong authentication if exposing API publicly
- Consider IP whitelisting in Cloudflare
- Monitor logs for unusual activity
- Use dedicated phone number for production

## 📱 Phone Number Formats

- **Individual**: `1234567890` or `+1234567890`
- **Groups**: Use group JID format `1234567890@g.us`
- **International**: Always include country code

## 🔄 Updates & Maintenance

```bash
# Update the application
cd Whatsapp-Private-Number-API
git pull origin main
cd whatsapp-bridge
go build -o whatsapp-api main.go
sudo systemctl restart whatsapp-api
```

## 📋 System Requirements

- **OS**: Ubuntu 20.04+ (ARM64 or AMD64)
- **RAM**: 512MB minimum, 1GB recommended
- **Storage**: 1GB available space
- **Network**: Stable internet connection
- **Go**: Version 1.21 or higher

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## 📄 License

This project is provided as-is for educational purposes. Use at your own risk and in compliance with WhatsApp's Terms of Service.

## ⚡ Quick Commands Reference

```bash
# Build and run
go build -o whatsapp-api main.go && ./whatsapp-api

# Service management
sudo systemctl start whatsapp-api
sudo systemctl restart whatsapp-api
sudo systemctl status whatsapp-api
sudo systemctl stop whatsapp-api

# Test API (only when service is running)
curl -X POST http://localhost:8080/api/send -H "Content-Type: application/json" -d '{"recipient":"NUMBER","message":"test"}'

# View logs
sudo journalctl -u whatsapp-api -f
```

---

**🚀 Ready to automate your WhatsApp communications!**

## 📱 Adding Multiple WhatsApp Numbers (Advanced)

You can run multiple WhatsApp APIs simultaneously, each connected to a different phone number on different ports and subdomains.

### Prerequisites for Multiple APIs

- Multiple phone numbers/SIM cards
- Each API runs on a different port (8080, 8081, 8082, etc.)
- Each API needs its own Cloudflare tunnel and subdomain 
- The new port needs to be added in the Cloud config as done before (🏗️ Oracle Cloud Server Setup) 

### Step-by-Step Setup for Second API

#### 1. Copy and Prepare Second API

```bash
# Copy the entire project for second API
cd /home/ubuntu
cp -r Whatsapp-Private-Number-API Whatsapp-Private-Number-API-2
cd Whatsapp-Private-Number-API-2/whatsapp-bridge

# Remove existing session data
rm -rf store/

# Clean up conflicting binaries from copy
rm whatsapp-api  # Remove original binary name
```

#### 2. Change Port Configuration

```bash
# Find and change port in code (should be line ~912)
grep -n "808" main.go
sed -i 's/8080/8081/g' main.go

# Verify change
grep -n "808" main.go  # Should show 8081 now

# Build new binary with different name
export CGO_ENABLED=1
go build -o whatsapp-api2 main.go
```

#### 3. Cloudflare Tunnel Setup (Windows)

**Important:** Tunnel creation must be done from Windows as Ubuntu servers are headless and cannot open browsers for authentication.

```powershell
# On Windows PowerShell
cd C:\Users\[YOUR_USERNAME]\.cloudflared

# Create second tunnel
.\cloudflared.exe tunnel create whatsapp-api2

# Set DNS for second subdomain  
.\cloudflared.exe tunnel route dns whatsapp-api2 whatsapp-api2.yourdomain.com
```

This will generate a new tunnel ID and credentials file.

#### 4. Ubuntu Cloudflare Configuration

```bash
# Create second tunnel config
sudo nano /etc/cloudflared/whatsapp-api2-config.yml
```

**Configuration content:**
```yaml
tunnel: whatsapp-api2
credentials-file: /home/ubuntu/.cloudflared/[NEW-TUNNEL-ID].json

ingress:
  - hostname: whatsapp-api2.yourdomain.com
    service: http://127.0.0.1:8081  # Note port 8081
  - service: http_status:404
```

```bash
# Copy credentials from Windows to Ubuntu
nano /home/ubuntu/.cloudflared/[NEW-TUNNEL-ID].json
# Paste the complete JSON content from Windows file
```

#### 5. Avoiding Service Conflicts

```bash
# Remove generic config if it exists (prevents conflicts)
sudo rm -f /etc/cloudflared/config.yml

# Create manual systemd service for second tunnel
sudo nano /etc/systemd/system/cloudflared-api2.service
```

**Service content:**
```ini
[Unit]
Description=Cloudflared API2 Tunnel
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/bin/cloudflared --config /etc/cloudflared/whatsapp-api2-config.yml tunnel run
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

#### 6. WhatsApp API2 System Service

```bash
# Create WhatsApp API2 service
sudo nano /etc/systemd/system/whatsapp-api2.service
```

**Service content:**
```ini
[Unit]
Description=WhatsApp API2 Service
After=network.target

[Service]
Type=simple
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/Whatsapp-Private-Number-API-2/whatsapp-bridge
ExecStart=/home/ubuntu/Whatsapp-Private-Number-API-2/whatsapp-bridge/whatsapp-api2
Restart=always
RestartSec=10
Environment=CGO_ENABLED=1

[Install]
WantedBy=multi-user.target
```

#### 7. Start All Services

```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable and start Cloudflare tunnel for API2
sudo systemctl enable cloudflared-api2
sudo systemctl start cloudflared-api2

# Enable and start WhatsApp API2
sudo systemctl enable whatsapp-api2
sudo systemctl start whatsapp-api2
```

#### 8. WhatsApp Authentication

```bash
# First run to get QR code for second phone number
cd /home/ubuntu/Whatsapp-Private-Number-API-2/whatsapp-bridge
./whatsapp-api2

# Scan QR code with your SECOND phone number
# After successful connection, Ctrl+C and restart service:
sudo systemctl restart whatsapp-api2
```

#### 9. Verify Both APIs

```bash
# Check all services
sudo systemctl status cloudflared        # API1 tunnel
sudo systemctl status cloudflared-api2   # API2 tunnel  
sudo systemctl status whatsapp-api       # API1 WhatsApp
sudo systemctl status whatsapp-api2      # API2 WhatsApp

# Test both APIs
curl -X POST https://whatsapp-api1.yourdomain.com/api/send \
  -H "Content-Type: application/json" \
  -d '{"recipient":"PHONE1","message":"Test from API1"}'

curl -X POST https://whatsapp-api2.yourdomain.com/api/send \
  -H "Content-Type: application/json" \
  -d '{"recipient":"PHONE2","message":"Test from API2"}'
```

### Multiple APIs Management

```bash
# Stop specific API
sudo systemctl stop whatsapp-api2
sudo systemctl stop cloudflared-api2

# Restart specific API
sudo systemctl restart whatsapp-api2
sudo systemctl restart cloudflared-api2

# View logs for specific API
sudo journalctl -u whatsapp-api2 -f
sudo journalctl -u cloudflared-api2 -f
```

### Important Notes for Multiple APIs

- **Each API needs a unique port** (8080, 8081, 8082, etc.)
- **Each API needs its own subdomain** (api1.domain.com, api2.domain.com)
- **Session data is isolated** - each API maintains its own `store/` directory
- **WhatsApp allows up to 4 linked devices** per phone number
- **Both APIs can run simultaneously** without interference
- **Each phone number maintains its own 14-day session timer**

### Scaling to More APIs

To add a third, fourth API, etc.:
1. Repeat the process with new ports (8082, 8083, etc.)
2. Create new tunnel names (whatsapp-api3, whatsapp-api4, etc.)
3. Use new subdomain names (whatsapp-api3.domain.com, etc.)
4. Ensure each has unique service names (whatsapp-api3, cloudflared-api3, etc.)

### Troubleshooting Multiple APIs

**Port Conflicts:**
```bash
# Check which ports are in use
sudo netstat -tulpn | grep 808

# Verify port in code
grep -n "808" /path/to/main.go
```

**Service Conflicts:**
```bash
# List all cloudflared services
sudo systemctl list-units | grep cloudflared

# Remove conflicting generic configs
sudo rm -f /etc/cloudflared/config.yml
```

**Authentication Issues:**
- Each API needs separate QR code scan with different phone numbers
- Sessions are independent - one API disconnecting doesn't affect others
- Check logs individually: `sudo journalctl -u whatsapp-api2 -f`