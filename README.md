# TradeNotifyEA v7 - Advanced Trading Notification System for MT5

## Overview

TradeNotifyEA is a comprehensive MetaTrader 5 Expert Advisor that provides real-time trade notifications via Telegram, complete with automated statistics tracking calculated directly from account history, chart screenshots, and detailed trade reporting. Designed for traders who want to monitor their trading activity remotely and maintain detailed performance records.

## Key Features

### 📱 Telegram Integration
- **Real-time notifications** for all trading activities
- **Automatic chart screenshots** attached to trade closure notifications
- **Thread support** for organized group chats (configurable topic ID)
- **Emoji-enhanced messages** for better readability

### 📊 Comprehensive Trade Tracking

#### Position Monitoring
- **Entry notifications** with full trade details:
  - Direction (Buy/Sell) with visual arrows (⬆/⬇)
  - Entry price
  - Take Profit with pip distance
  - Stop Loss with pip distance
  - Lot size management (half lot vs full lot based on configurable barrier)
  - Custom entry notes

- **Exit notifications** for:
  - Partial closures (50% position reduction)
  - Complete closures
  - Profit/loss in both percentage and pips
  - Automatic chart screenshot on full closure

- **SL/TP modification alerts**:
  - Real-time updates when stop loss or take profit levels change
  - Pip distance calculations from entry
  - Visual comparison of old vs new levels

#### Pending Order Management
- **Order placement notifications** with complete details
- **Order modification tracking**:
  - Price changes
  - SL/TP updates
  - Smart detection of first SL addition
- **Order cancellation alerts** with reason (canceled, rejected, expired, failed)
- **Failed order notifications** with return codes and error messages

### 📈 Advanced Statistics System

#### Multi-Timeframe Reporting
- **Daily statistics** (Monday-Friday at 21:00)
- **Weekly statistics** (Friday at 21:00)
- **Monthly statistics** (last day of month at 21:00)

#### Statistical Metrics
- Total number of trades
- Total pips gained/lost (with proper symbol-specific calculations)
- Total profit/loss percentage (based on actual trading results)
- Average per-trade statistics (weekly/monthly)
- Balance tracking (start vs end balance for weekly reports)

#### Calculation Method
- **No binary files** - all statistics calculated in real-time from MT5 history
- **Accurate balance calculations** - tracks starting balance by working backwards from current balance
- **Precise pip calculations** - handles different symbol types (Forex, Gold, JPY pairs)
- **True trading profit** - excludes balance operations, includes commissions and swaps

#### CSV Export
Historical data saved automatically:
- `TradeNotify_Daily_History.csv`
- `TradeNotify_Weekly_History.csv`
- `TradeNotify_Monthly_History.csv`

### 📸 Chart Screenshot Capabilities
- **Automatic screenshots** on complete position closure
- **Configurable dimensions** (default: 880x820 pixels)
- **M1 timeframe** for detailed view
- **Automatic cleanup** of screenshots at 21:00 daily
- **Smart chart detection** or temporary chart creation

## Configuration Parameters

### Telegram Settings
```mql5
input string TELEGRAM_BOT_TOKEN = "your_bot_token_here";
input string TELEGRAM_CHAT_ID = "your_chat_id_here";
input string TELEGRAM_TOPIC_ID = "your_channel_id_here"; 
```

### Message Customization
```mql5
input double LOT_SIZE_BARRIER = 0.02; // Threshold for "half lot" vs "full lot" labeling
input string NOTE_ENTRY_SIGNAL = "Use proportional lot size with your own account balance";
input string NOTE_EXIT_PARTIAL = "Am inchis jumatate din pozitie, urmeaza inchiderea totala";
input string NOTE_EXIT_TOTAL = "Am inchis restul pozitiei";
```

### Statistics Preferences
```mql5
input bool SEND_DAILY_STATS = true;
input bool SEND_WEEKLY_STATS = true;
input bool SEND_MONTHLY_STATS = true;
```

## How It Works

### Initialization Process
1. **Validates Telegram credentials** on startup
2. **Prepares state tracking** for positions and pending orders
3. **Ready for real-time monitoring** - no file loading needed

### Real-Time Monitoring

#### Position Lifecycle Tracking
The EA uses the `OnTradeTransaction()` event handler to monitor:

1. **TRADE_TRANSACTION_DEAL_ADD**: 
   - Detects new position entries (DEAL_ENTRY_IN)
   - Captures partial/full closures (DEAL_ENTRY_OUT)
   - Automatically retrieves live position data for accuracy

2. **Position State Management**:
   - Maintains internal array of `PositionState` structures
   - Tracks: position ID, symbol, direction, entry price, SL, TP, initial volume
   - Monitors partial closure flag to avoid duplicate notifications

3. **Smart Partial Closure Detection**:
   - Compares remaining volume to initial volume
   - Triggers partial notification when volume ≤ 50% of initial
   - Ensures single notification per partial closure

#### Pending Order Tracking
1. **ORDER_ADD**: Captures new pending orders (only notifies if SL is set)
2. **ORDER_UPDATE**: Detects modifications (price, SL, TP changes)
   - Smart detection of first SL addition (triggers full entry message)
   - Ignores updates if no SL is set
3. **ORDER_DELETE**: Identifies cancellations/rejections/expirations
4. **Duplicate prevention**: Tracks notified deletions to avoid spam
5. **Failed order notifications**: Captures failed placement requests with error details

### Statistics Calculation

#### Automated Scheduling
- **OnTick() scheduler** checks time every tick
- **Executes once per day** using `last_report_date` guard
- **21:00 local time** trigger for all reports

#### Advanced Calculation Logic

**Balance Calculation**:
```
Start Balance = Current Balance - Sum of all trades/swaps/commissions since start time
```

**Pip Calculation** (symbol-specific):
- Gold (XAU): 0.10 pip size
- JPY pairs: 0.01 pip size
- Standard Forex: 0.0001 pip size
- Formula: (exit_price - entry_price) / pip_size (adjusted for direction)

**Profit Calculation**:
```
Trading Profit = Sum(deal_profit + commission + swap) excluding balance operations
Profit % = (Trading Profit / Start Balance) × 100
```

**Entry-Exit Matching**:
- EA stores all DEAL_ENTRY_IN deals
- Matches DEAL_ENTRY_OUT deals to corresponding entries by position ID
- Calculates pips for each complete trade
- Prevents double-counting of same position

#### Report Schedule
- **Daily stats**: Monday-Friday at 21:00
- **Weekly stats**: Friday at 21:00 (with start/end balance)
- **Monthly stats**: Last day of month at 21:00

### Message Formatting

#### Entry Signal Example
```
ℹ Flaviu trade idea
#EURUSD - Buy NOW!
⬆ Lot Management: full lot
⬆ Entry Price: 1.08500
⬆ TakeProfit: 1.08650 ( 15.0 pips )
⬆ StopLoss: 1.08400 ( 10.0 pips )
Note - Use proportional lot size with your own account balance
```

#### SL/TP Update Example
```
ℹ Flaviu trade idea
#EURUSD - ⬆ Buy NOW!
⬆ Entry Price: 1.08500
⬆ TakeProfit: 1.08650 ( 15.0 pips )
⬆ StopLoss: 1.08450 ( 5.0 pips )
Note - Use proportional lot size with your own account balance
```

#### Exit Signal Example
```
❌ Flaviu trade idea
#EURUSD close total
❌ Exit Price: 1.08650 ( +15.0 pips )
❌ Profit Made: 1.25%
Note - Am inchis restul pozitiei
```

#### Pending Order First SL Addition
```
ℹ Flaviu trade idea
#EURUSD - ⬆ Buy Stop
⬆ Lot Management: full lot
⬆ Entry Price: 1.08500
⬆ TakeProfit: 1.08650 ( 15.0 pips )
⬆ StopLoss: 1.08400 ( 10.0 pips )
Note - Use proportional lot size with your own account balance
```

#### Statistics Report Example
```
📊 Flaviu Trading Statistics - This Week

📈 Total Trades: 12
📊 Total Pips: +145.50
💰 Total Profit: +3.25%

📊 Average per trade:
• Pips: 12.12
• Profit: 0.27%
```

## Technical Implementation Details

### State Management
- **Dynamic arrays** for position and order states
- **Helper functions** for state lookup, addition, and removal
- **Retry mechanism** (up to 5 attempts) for retrieving live position data

### History-Based Calculations
- **HistorySelect()** used to retrieve deal history for time periods
- **Deal filtering** excludes balance operations (deposits/withdrawals)
- **Backward balance calculation** for accurate starting balance
- **Commission and swap inclusion** for true profit figures

### Symbol-Specific Handling
- **Automatic pip size detection** based on symbol name
- **Gold/XAU**: 0.10 pip size
- **JPY pairs**: 0.01 pip size
- **All others**: 0.0001 pip size

### Error Handling
- **WebRequest error checking** with detailed logging
- **File operation validation** for CSV exports and screenshots
- **Fallback mechanisms** for missing data
- **Retry loops** for history access and order deletion detection

### URL Encoding
- **Custom implementation** for proper Telegram API parameter encoding
- **UTF-8 support** for international characters
- **Special character handling** (spaces, newlines, etc.)

### Multipart Upload
- **Custom HTTP multipart/form-data** implementation for photo uploads
- **Boundary generation** using tick count
- **Fallback to document upload** if photo upload fails

### Screenshot Cleanup
- **Pattern matching** for today's screenshots
- **Multiple file naming patterns** supported:
  - `.raw_YYYYMMDD_*`
  - `.r_YYYYMMDD_*`
  - `.std_YYYYMMDD_*`
- **Automatic deletion** at 21:00 daily

## Setup Instructions

### 1. Create Telegram Bot
1. Message [@BotFather](https://t.me/botfather) on Telegram
2. Send `/newbot` and follow instructions
3. Copy your bot token (format: `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`)

### 2. Get Chat ID
**For private chat:**
1. Message your bot
2. Visit: `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`
3. Find your `chat_id` in the response

**For group chat:**
1. Add bot to group
2. Send a message in the group
3. Visit same URL as above
4. Find `chat.id` (will be negative number)

**For topic/thread:**
1. Right-click thread → Copy link
2. Thread ID is the last number in URL

### 3. MetaTrader 5 Configuration
1. **Enable WebRequest** for Telegram API:
   - Tools → Options → Expert Advisors
   - Check "Allow WebRequest for listed URL"
   - Add: `https://api.telegram.org`

2. **Compile and attach EA**:
   - Place `.mq5` file in `MQL5/Experts/` folder
   - Open MetaEditor (F4) and compile
   - Drag EA onto any chart

3. **Configure parameters**:
   - Enter your `TELEGRAM_BOT_TOKEN`
   - Enter your `TELEGRAM_CHAT_ID`
   - (Optional) Enter `TELEGRAM_TOPIC_ID` for threads
   - Customize messages and statistics preferences

### 4. Test Installation
You can manually test the EA by:
- Opening a demo position
- Checking for Telegram notification
- Viewing EA logs in "Experts" tab
- Waiting for 21:00 to see statistics (or modify code for testing)

## File Structure

### Generated Files
```
MQL5/Files/
├── TradeNotify_Daily_History.csv         # Daily CSV export
├── TradeNotify_Weekly_History.csv        # Weekly CSV export
├── TradeNotify_Monthly_History.csv       # Monthly CSV export
└── [SYMBOL]_[TIMESTAMP].png              # Chart screenshots (auto-deleted)
```

### CSV Formats

**Daily CSV**:
```csv
Date,Total_Trades,Total_Pips,Profit_Percentage
2025-01-15,5,+45.50,+2.35
```

**Weekly CSV**:
```csv
Date,Total_Trades,Total_Pips,Profit_Percentage,Start_Balance,End_Balance
2025-01-17,12,+145.50,+3.25,10000.00,10325.00
```

**Monthly CSV**:
```csv
Date,Total_Trades,Total_Pips,Profit_Percentage
2025-01-31,48,+520.30,+12.45
```

## Performance Considerations

- **Lightweight monitoring**: Uses event-driven architecture (no polling)
- **Efficient state management**: Minimal memory footprint with dynamic arrays
- **History-based calculations**: No persistent files to manage
- **Screenshot cleanup**: Automatic deletion prevents storage bloat
- **Rate limiting aware**: Single notification per event

## Limitations & Notes

- **Requires internet connection** for Telegram notifications
- **MT5 platform must be running** for notifications to be sent
- **WebRequest must be enabled** in MT5 settings
- **Screenshots only on full closure** (not partial)
- **Statistics reset schedule is fixed** (21:00 local time)
- **History-based calculations** depend on MT5 history availability
- **7-day history window** for calculations (configurable in code)
- **Pip calculations are estimations** for symbols with non-standard pip sizes

## Troubleshooting

### No notifications received
1. Check MT5 "Experts" tab for errors
2. Verify WebRequest is enabled for `api.telegram.org`
3. Confirm bot token and chat ID are correct
4. Test bot by sending `/start` command directly

### Statistics showing zero
1. Ensure trades exist in the history for the period
2. Check MT5 "Account History" tab to verify deals are present
3. Review EA logs for `HistorySelect` errors
4. Verify account has trading history (not brand new account)

### Incorrect pip calculations
1. Verify symbol naming (EA uses string matching for special symbols)
2. Check if symbol is non-standard (may need custom pip size in code)
3. Review "Experts" tab logs for calculation details

### Screenshot failures
1. Ensure chart is available for symbol
2. Check if symbol data is loaded
3. Review EA logs for `ChartScreenShot` errors
4. Verify sufficient disk space in MQL5/Files folder

### CSV files not saving
1. Check file permissions in `MQL5/Files/` folder
2. Review MT5 logs for file operation errors
3. Verify sufficient disk space
4. Ensure EA has write permissions

## Advanced Configuration

### Modifying Report Times
To change the 21:00 report time, edit the `OnTick()` function:
```mql5
if(dt.hour == 21 && dt.min >= 0 && dt.min <= 1 && last_report_date != today)
```
Change `dt.hour == 21` to your desired hour (24-hour format).

### Customizing Pip Sizes
To add custom pip sizes for specific symbols, modify the `CalculateTradesAndPips()` function:
```mql5
double pip_size;
if(StringFind(symbol, "YOUR_SYMBOL") >= 0)
{
   pip_size = 0.01; // Your custom pip size
}
```

### Adjusting History Window
To change the 7-day history window, modify:
```mql5
HistorySelect(now-7*24*3600,now);
```
Change `7` to desired number of days.

## Version History

**v7.0** - Current Release
- Removed binary statistics file dependency
- Implemented real-time history-based calculations
- Added accurate balance tracking
- Improved pip calculations for different symbol types
- Enhanced CSV export with balance information
- Better screenshot cleanup with pattern matching
- Added failed order notifications
- Improved SL/TP modification detection

## License

Copyright 2025, MetaQuotes Ltd.  
https://www.mql5.com

---

**Author Notes**: This EA is designed for traders who value transparency and detailed record-keeping. All statistics are calculated directly from your account history, ensuring accuracy without relying on persistent storage files. All notifications are sent in real-time, ensuring you never miss a trading opportunity or position change, even when away from your terminal.
