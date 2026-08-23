## MT5 Setup Guide

This guide shows you how to configure the Trading Signals API with MetaTrader5.

### Prerequisites

1. **MetaTrader5 Terminal** (Windows only)
   - Download from: https://www.metatrader5.com/
   - Install and complete setup

2. **Python Environment**
   - Python 3.8 or higher
   - All dependencies installed: `pip install -r requirements.txt`

### Step 1: Enable MT5 API Access

1. Open MetaTrader5 terminal
2. Go to **Tools → Options**
3. Click on **Expert Advisors** tab
4. Enable these options:
   - ✓ Allow automated trading
   - ✓ Allow DLL imports
   - ✓ Allow WebRequest for listed URL
5. Click **OK**

### Step 2: Get Your MT5 Credentials

#### For Demo Account

1. In MT5, go to **File → Open an Account**
2. Select your broker (e.g., FTMO)
3. Choose **Demo Account**
4. Complete registration
5. Note your credentials:
   - Account Number (login)
   - Password
   - Server Name

#### For Live/Challenge Account

1. Log into your broker's client portal
2. Find your MT5 account details
3. Note your credentials:
   - Account Number (login)
   - Password
   - Server Name

### Step 3: Configure Environment Variables

#### Option A: Using .env File (Recommended)

1. Copy the template:
   ```bash
   cp .env.template .env
   ```

2. Edit `.env` with your credentials:
   ```bash
   # Enable MT5
   USE_MT5=true

   # Your MT5 credentials
   MT5_ACCOUNT=your_account_id
   MT5_PASSWORD=your_password_here
   MT5_SERVER=FTMO-Demo
   MT5_ACCOUNT_TYPE=DEMO
   ```

3. **Important**: Add `.env` to `.gitignore` to avoid committing credentials

#### Option B: Using System Environment Variables

##### Linux/Mac
```bash
export USE_MT5=true
export MT5_ACCOUNT=your_account_id
export MT5_PASSWORD=your_password
export MT5_SERVER=FTMO-Demo
export MT5_ACCOUNT_TYPE=DEMO
```

##### Windows Command Prompt
```cmd
set USE_MT5=true
set MT5_ACCOUNT=your_account_id
set MT5_PASSWORD=your_password
set MT5_SERVER=FTMO-Demo
set MT5_ACCOUNT_TYPE=DEMO
```

##### Windows PowerShell
```powershell
$env:USE_MT5="true"
$env:MT5_ACCOUNT="your_account_id"
$env:MT5_PASSWORD="your_password"
$env:MT5_SERVER="FTMO-Demo"
$env:MT5_ACCOUNT_TYPE="DEMO"
```

### Step 4: Verify MT5 Connection

Before starting the API, verify your MT5 connection:

```python
import MetaTrader5 as mt5

# Initialize MT5
if not mt5.initialize():
    print(f"MT5 initialization failed: {mt5.last_error()}")
    quit()

# Login
authorized = mt5.login(
    login=your_account_id,
    password="your_password",
    server="FTMO-Demo"
)

if not authorized:
    print(f"Login failed: {mt5.last_error()}")
    mt5.shutdown()
    quit()

# Get account info
account = mt5.account_info()
print(f"Connected to: {account.server}")
print(f"Balance: ${account.balance}")

mt5.shutdown()
```

### Step 5: Start the API

#### Using the run script:
```bash
python run_api.py
```

#### Or directly:
```bash
python trading_signals_api.py
```

#### Check the logs:

Look for these messages:

```
✓ Loaded configuration from .env
MT5 connected successfully
Account: your_account_id
Server: FTMO-Demo
Balance: $100,000.00
API started with MT5 integration
```

### Step 6: Test the Connection

```bash
# Test the API
python test_api.py

# Or use curl
curl http://localhost:8000/account
```

Expected response (connected):
```json
{
  "connected": true,
  "login": your_account_id,
  "server": "FTMO-Demo",
  "balance": 100000.00,
  "equity": 100000.00,
  "account_type": "DEMO"
}
```

### Common FTMO Server Names

#### Demo Accounts
- `FTMO-Demo` - Free trial accounts

#### Challenge/Verification Accounts
- `FTMO-Server`
- `FTMO-Server2`
- `FTMO-Server3`
- `FTMO-Server4`

#### Funded Accounts
- `FTMO-Server-Real`

### Troubleshooting

#### "MT5 initialization failed"

**Cause**: MT5 terminal not running or API access disabled

**Solution**:
1. Start MetaTrader5 terminal
2. Enable API access in Options → Expert Advisors
3. Restart the terminal

#### "MT5 login failed"

**Cause**: Incorrect credentials or server name

**Solution**:
1. Verify account number is correct
2. Check password (case-sensitive)
3. Confirm server name matches your account
4. Ensure account is not expired

#### "MetaTrader5 library not available"

**Cause**: MT5 Python library not installed

**Solution**:
```bash
pip install MetaTrader5
```

**Note**: MetaTrader5 library only works on Windows

#### API shows "Using simulated data"

**Cause**: MT5 connection not established

**Check**:
1. `USE_MT5=true` in environment
2. MT5 credentials are correct
3. MT5 terminal is running
4. Check API logs for specific error

#### Connection drops during operation

**Cause**: MT5 terminal closed or network issues

**Solution**:
1. Keep MT5 terminal running
2. Check network connection
3. API will auto-fallback to simulated data

### Security Best Practices

1. **Never commit credentials**
   ```bash
   # Add to .gitignore
   echo ".env" >> .gitignore
   ```

2. **Use environment variables in production**
   - Set via system/container environment
   - Never hardcode credentials

3. **Use separate demo accounts for testing**
   - Don't use live account credentials for testing
   - Create dedicated demo accounts

4. **Rotate passwords regularly**
   - Change MT5 passwords periodically
   - Update .env file accordingly

5. **Restrict API access**
   - Use firewall rules
   - Implement authentication if exposing publicly

### Running Without MT5

If you don't have MT5 or want to test without it:

```bash
# Set in .env or environment
USE_MT5=false
```

The API will work with simulated market data, perfect for:
- Development and testing
- Demonstrations
- Learning the API
- Non-Windows environments

### Next Steps

1. Test signal generation: `curl "http://localhost:8000/signals?symbol=EURUSD"`
2. Review API documentation: `http://localhost:8000/docs`
3. Integrate with your trading system
4. Monitor logs for any issues

### Support

For issues:
1. Check the logs
2. Verify MT5 terminal is running
3. Test credentials in MT5 directly
4. Review troubleshooting section above