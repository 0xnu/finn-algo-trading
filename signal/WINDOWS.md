## Windows EC2 Instance Configuration

To access your FastAPI endpoints from a [Windows EC2](https://aws.amazon.com/windows/products/ec2/) instance using the public IP, you need to configure several things. Here's the complete breakdown:

### 1. **Security Group Configuration (AWS)**

First, allow inbound traffic to port 8000:

1. Go to AWS EC2 Console
2. Select your Windows instance
3. Click on the **Security** tab
4. Click on the security group link
5. Click **Edit inbound rules**
6. Add a new rule:
   - **Type**: Custom TCP
   - **Port Range**: 8000
   - **Source**: 
     - `0.0.0.0/0` (allow from anywhere - **not recommended for production**)
     - Or your specific IP: `YOUR_IP/32` (more secure)
   - **Description**: FastAPI Trading Signals
7. Click **Save rules**

### 2. **Windows Firewall Configuration**

On your Windows EC2 instance, allow port 8000:

#### Using Windows Defender Firewall GUI:
1. Open **Windows Defender Firewall with Advanced Security**
2. Click **Inbound Rules** → **New Rule**
3. Select **Port** → **Next**
4. Select **TCP** and enter **8000** → **Next**
5. Select **Allow the connection** → **Next**
6. Check all profiles (Domain, Private, Public) → **Next**
7. Name it "FastAPI Port 8000" → **Finish**

#### Using PowerShell (faster):
```powershell
New-NetFirewallRule -DisplayName "FastAPI Port 8000" -Direction Inbound -LocalPort 8000 -Protocol TCP -Action Allow
```

#### Using Command Prompt:
```cmd
netsh advfirewall firewall add rule name="FastAPI Port 8000" dir=in action=allow protocol=TCP localport=8000
```

### 3. **Start the API Server**

The current code already binds to `0.0.0.0`, which is correct:

```python
uvicorn.run(app, host="0.0.0.0", port=8000)
```

This means it listens on all network interfaces, not just localhost.

Start the server:
```cmd
python trading_signals_api.py
```

Or:
```cmd
python run_api.py
```

### 4. **Access the API**

#### From Your Local Machine:

Replace `YOUR_EC2_PUBLIC_IP` with your actual EC2 public IP address:

```bash
# Test health endpoint
curl http://YOUR_EC2_PUBLIC_IP:8000/health

# Get account info
curl http://YOUR_EC2_PUBLIC_IP:8000/account

# Generate signals
curl http://YOUR_EC2_PUBLIC_IP:8000/signals?symbol=EURUSD

# View API documentation
# Open in browser:
http://YOUR_EC2_PUBLIC_IP:8000/docs
```

#### Example with Real IP:
```bash
# If your EC2 public IP is <EC2_PUBLIC_IP>
curl http://<EC2_PUBLIC_IP>:8000/health

# In browser
http://<EC2_PUBLIC_IP>:8000/docs
```

### 5. **Verify Your EC2 Public IP**

Get your EC2 public IP:

#### From AWS Console:
- EC2 Dashboard → Instances → Select your instance
- Look for **Public IPv4 address** or **Public IPv4 DNS**

#### From Windows EC2 Instance:
```powershell
# PowerShell
Invoke-RestMethod -Uri http://checkip.amazonaws.com

# Or
(Invoke-WebRequest -Uri "http://ifconfig.me").Content
```

```cmd
# Command Prompt
curl http://checkip.amazonaws.com
```

### 6. **Test Connection from EC2 Instance Itself**

Before testing externally, verify locally on the EC2 instance:

```powershell
# Test from the EC2 instance itself
Invoke-RestMethod -Uri http://localhost:8000/health

# Or
curl http://localhost:8000/health
```

### 7. **Production Best Practices**

#### A. Use HTTPS with SSL/TLS

Instead of exposing port 8000 directly, use a reverse proxy:

**Option 1: NGINX (recommended)**
1. Install NGINX on Windows
2. Configure SSL certificate (Let's Encrypt)
3. Proxy requests to localhost:8000
4. Open port 443 (HTTPS) in security group

**Option 2: Use AWS Application Load Balancer**
1. Create ALB with SSL certificate
2. Target your EC2 instance on port 8000
3. Only allow ALB security group to access port 8000

#### B. Restrict Access by IP

Instead of `0.0.0.0/0`, whitelist specific IPs:

In AWS Security Group:
```
Source: 203.0.113.0/32  # Your office IP
Source: 198.51.100.0/24  # Your company network
```

#### C. Add Authentication

Modify the API to require API keys or JWT tokens.

## 8. **Troubleshooting**

#### Cannot connect from outside:

**Check 1: Security Group**
```bash
# From your local machine
telnet YOUR_EC2_PUBLIC_IP 8000
```
If this fails, security group is blocking.

**Check 2: Windows Firewall**
```powershell
# On EC2 instance - check if rule exists
Get-NetFirewallRule -DisplayName "FastAPI Port 8000"
```

**Check 3: API is running**
```powershell
# On EC2 instance
netstat -an | findstr :8000
```
Should show: `0.0.0.0:8000` or `[::]​:8000`

**Check 4: Correct binding**
Look at the uvicorn startup logs:
```
INFO:     Uvicorn running on http://0.0.0.0:8000
```
If it says `http://127.0.0.1:8000`, it won't accept external connections.

#### Server stops when you close RDP:

Run as a Windows Service or use:

**Option 1: nohup equivalent**
```powershell
Start-Process python -ArgumentList "trading_signals_api.py" -NoNewWindow -RedirectStandardOutput "api.log" -RedirectStandardError "error.log"
```

**Option 2: Task Scheduler**
Create a scheduled task to run at startup.

**Option 3: Windows Service (best for production)**
Use `nssm` (Non-Sucking Service Manager):
```cmd
nssm install TradingSignalsAPI "C:\Python\python.exe" "C:\path\to\trading_signals_api.py"
nssm start TradingSignalsAPI
```

### 9. **Complete Example**

```bash
# 1. Get your EC2 public IP (run on EC2)
curl http://checkip.amazonaws.com
# Output: <EC2_PUBLIC_IP>

# 2. Configure AWS Security Group
# Add inbound rule: TCP port 8000 from 0.0.0.0/0

# 3. Configure Windows Firewall (run on EC2)
netsh advfirewall firewall add rule name="FastAPI Port 8000" dir=in action=allow protocol=TCP localport=8000

# 4. Start API (run on EC2)
python trading_signals_api.py

# 5. Test from your local machine
curl http://<EC2_PUBLIC_IP>:8000/health

# 6. Open in browser
http://<EC2_PUBLIC_IP>:8000/docs
```

The code doesn't need any changes; it's already configured correctly with `host="0.0.0.0"`. You just need to configure the network and firewall settings.