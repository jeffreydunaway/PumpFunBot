# PumpFun Bot

A production-ready Solana token monitoring and trading bot for [PumpFun](https://pump.fun).

## Features

- **Real-time Token Monitoring**: Polls PumpFun API for new token launches and migrations
- **Security Analysis**: Integrates with RugCheck and GoPlus for comprehensive token safety checks
- **Smart Filtering**: Configurable filters for liquidity, holder count, creator reputation, and more
- **Paper Trading**: Test strategies without risking real funds
- **Live Trading**: Execute swaps via Jupiter aggregator (use with caution)
- **Telegram Integration**: Receive alerts and execute trades via Telegram bot
- **Position Management**: Stop-loss, take-profit, and trailing stop orders
- **Blacklist Management**: Block known scam tokens and creators
- **Persistent Storage**: SQLite (development) or PostgreSQL (production)

## Architecture

```
pumpfun_bot/
├── config/           # Configuration management (Pydantic settings)
├── core/             # Main bot orchestrator
├── models/           # Database models and DTOs
├── services/         # Business logic services
│   ├── database.py   # Async database operations
│   ├── pumpfun_api.py # PumpFun API client
│   ├── security.py   # Security analysis (RugCheck, GoPlus)
│   ├── trading.py    # Trade execution (paper + live)
│   └── telegram_bot.py # Telegram bot interface
└── utils/            # Utilities (logging, helpers)
```

## Quick Start

### Prerequisites

- Python 3.11+
- Solana RPC endpoint (free tier from Helius, QuickNode, etc.)
- Telegram bot token (optional, from @BotFather)

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/pumpfun-bot.git
cd pumpfun-bot

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -e ".[all]"

# Copy and configure environment
cp .env.example .env
# Edit .env with your settings
```

### Configuration

Edit `.env` with your settings:

```env
# Required
PUMPFUN_API_SOLANA_RPC_URL=https://your-rpc-endpoint.com

# For Telegram alerts (optional but recommended)
PUMPFUN_TELEGRAM_BOT_TOKEN=your_bot_token
PUMPFUN_TELEGRAM_CHANNEL_ID=your_channel_id
PUMPFUN_TELEGRAM_ADMIN_USER_IDS=your_user_id

# Security (strongly recommended)
PUMPFUN_API_RUGCHECK_API_KEY=your_key  # Optional but improves results
```

### Running

```bash
# Initialize database
python -m pumpfun_bot --init-db

# Run in paper trading mode (default, safe)
python -m pumpfun_bot

# Run with debug logging
python -m pumpfun_bot --debug

# Run in live trading mode (CAUTION!)
python -m pumpfun_bot --live
```

## Telegram Commands

### Public Commands
- `/start` - Welcome message and command list
- `/status` - Bot status and connection info
- `/balance` - Current SOL balance
- `/portfolio` - Portfolio overview and positions
- `/analyze <token>` - Security analysis for a token

### Admin Commands (restricted to admin user IDs)
- `/buy <amount> <token>` - Execute buy order
- `/sell <amount> <token>` - Execute sell order

## Security Analysis

The bot performs multi-layer security checks:

1. **Blacklist Check**: Fast-fail for known scam tokens/creators
2. **RugCheck Integration**: Score, verdict, and risk flags
3. **GoPlus Security**: Honeypot detection, tax analysis
4. **Creator Reputation**: Track creator history and behavior
5. **Token Metrics**: Liquidity, holder distribution, age

Tokens must pass all checks before appearing in alerts.

## Filter Configuration

```env
# Liquidity range (SOL)
PUMPFUN_FILTER_MIN_LIQUIDITY_SOL=5.0
PUMPFUN_FILTER_MAX_LIQUIDITY_SOL=1000.0

# Holder requirements
PUMPFUN_FILTER_MIN_HOLDERS=25
PUMPFUN_FILTER_MAX_TOP_HOLDER_PERCENT=20.0

# Creator limits
PUMPFUN_FILTER_MAX_CREATOR_TOKENS=3

# Tax limits
PUMPFUN_FILTER_MAX_BUY_TAX_PERCENT=10.0
PUMPFUN_FILTER_MAX_SELL_TAX_PERCENT=10.0
```

## Risk Management

```env
# Position sizing
PUMPFUN_TRADING_DEFAULT_BUY_AMOUNT_SOL=0.1
PUMPFUN_TRADING_MAX_POSITION_SOL=1.0

# Automatic exits
PUMPFUN_TRADING_STOP_LOSS_PERCENT=15.0
PUMPFUN_TRADING_TAKE_PROFIT_PERCENT=50.0
PUMPFUN_TRADING_TRAILING_STOP_PERCENT=10.0

# Execution
PUMPFUN_TRADING_SLIPPAGE_BPS=150  # 1.5%
```

## Production Deployment

### With Docker

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY . .

RUN pip install --no-cache-dir -e ".[postgres]"

CMD ["python", "-m", "pumpfun_bot"]
```

### With systemd

```ini
[Unit]
Description=PumpFun Trading Bot
After=network.target

[Service]
Type=simple
User=pumpfun
WorkingDirectory=/home/pumpfun/pumpfun-bot
ExecStart=/home/pumpfun/pumpfun-bot/venv/bin/python -m pumpfun_bot
Restart=always
RestartSec=10
Environment=PUMPFUN_ENVIRONMENT=production

[Install]
WantedBy=multi-user.target
```

### Production Checklist

- [ ] Use dedicated RPC endpoint (not public)
- [ ] Use PostgreSQL for database
- [ ] Enable structured JSON logging
- [ ] Set up monitoring/alerting
- [ ] Use dedicated trading wallet with limited funds
- [ ] Test thoroughly in paper mode first
- [ ] Set conservative position limits
- [ ] Enable stop-loss on all positions

## ⚠️ Disclaimer

**USE AT YOUR OWN RISK.**

- This software is provided "as is" without warranty
- Cryptocurrency trading involves substantial risk of loss
- Past performance does not guarantee future results
- Never invest more than you can afford to lose
- The authors are not responsible for any financial losses
- Always start with paper trading to understand the system

## Development

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Run tests
pytest

# Format code
black pumpfun_bot
isort pumpfun_bot

# Type checking
mypy pumpfun_bot

# Lint
ruff check pumpfun_bot
```

## License

MIT License - see [LICENSE](LICENSE) for details.

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Write tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

## Support

- Open an issue for bugs or feature requests
- Star the repo if you find it useful
