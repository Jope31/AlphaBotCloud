# **Alpha Bot: Volatility-Adjusted Trailing Stop for Composer.trade**

Alpha Bot is a stateful, algorithmic trading companion script designed to sit on top of your [Composer.trade](https://www.composer.trade/) portfolio.

Instead of relying on rigid, pre-programmed exit rules, Alpha Bot uses live Monte Carlo simulations and dynamically calculates the Normalized Average True Range (NATR) of your specific holdings to implement a highly intelligent, volatility-adjusted trailing stop.

## **🧠 How It Works**

Alpha Bot operates entirely locally and acts as an intraday guardian for your symphonies:

1. **The Intraday Memory (High Water Mark):** The bot continuously tracks the live returns of your symphonies. It records the highest profit point reached during the day (the "High Water Mark") and saves it to a local bot\_state.json file.  
2. **The Arming Mechanism (Monte Carlo):** Every time the bot wakes up, it looks at the current day's S\&P 500 (SPY) performance and finds the 150 most similar historical trading days over the last 3 years using Alpaca data. It then runs 5,000 Monte Carlo paths. If the probability of the symphony closing higher than its current return drops below a defined threshold (e.g., 15%), the bot enters an **ARMED** state.  
3. **The Volatility Leash (NATR):** Once armed, the bot calculates how much your specific holdings *normally* swing in a day (the NATR). It multiplies this by a baseline factor (e.g., 2.0x) to set a dynamic trailing stop.  
   * *High volatility assets get a wider leash.*  
   * *Low volatility assets get a tighter leash.*  
4. **The Profit Parachute:** If a symphony goes on a massive run and its intraday peak exceeds its normal daily volatility, the bot automatically begins shrinking the trailing stop multiplier. The higher it flies, the tighter the leash becomes, violently protecting outlier profits.  
5. **The Red Day Defense:** If the market gaps down overnight, the bot automatically switches to a defensive posture, applying a much tighter multiplier (e.g., 0.75x) to cut losses quickly on weak opens.  
6. **Execution:** If the live return falls below the dynamic trailing stop distance from the High Water Mark, the bot executes a "Sell all to Cash" command via the Composer API and fires a Discord alert.

## **🛠️ Prerequisites & Setup**

1. **Python 3.13+** installed on your system.  
2. Clone this repository and install the required dependencies:  
   pip install requests numpy python-dotenv

3. Create a .env file in the root directory with your API credentials:

\# API Credentials  
COMPOSER\_KEY\_ID=your\_composer\_key\_here  
COMPOSER\_SECRET=your\_composer\_secret\_here  
ACCOUNT\_UUIDS=uuid\_1,uuid\_2

ALPACA\_KEY=your\_alpaca\_key\_here  
ALPACA\_SECRET=your\_alpaca\_secret\_here

DISCORD\_WEBHOOK\_URL=\[https://discord.com/api/webhooks/\](https://discord.com/api/webhooks/)...

\# Algorithm Parameters  
TRIGGER\_THRESHOLD\_PCT=15.0  
ATR\_LOOKBACK\_DAYS=14  
BASE\_ATR\_MULTIPLIER=2.0  
RED\_DAY\_ATR\_MULTIPLIER=0.75  
MIN\_MULTIPLIER\_FLOOR=0.5

## **📊 Example Output**

### **Console Log (Tracking Mode)**

Fetching 3-year history from Alpaca for 12 tickers in batches...  
  \-\> History download complete.  
Market Conditioning Baseline (SPY on 2026-04-02): 0.45%  
Market Open Tone: GREEN (Gap: 0.12%)

Evaluating Account: a1b2c3d4-xxxx-xxxx  
  \-\> Tech Momentum Core: Live Return \= 1.25% | High Water Mark \= 1.25% | Prob Beating \= 42.1%  
  \-\> Copy of Nova Feaver FR: Live Return \= 0.31% | High Water Mark \= 0.31% | Prob Beating \= 14.8%  
  \*\*\* WARNING: Copy of Nova Feaver FR ARMED. Monte Carlo Probability dropped below threshold. \*\*\*  
  \-\> \[ARMED\] Stop Distance: 2.45% | Current Drawdown: 0.00%  
