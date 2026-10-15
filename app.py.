from datetime import datetime
import time
import pandas as pd
import streamlit as st
import yfinance as yf

# Auto-refresh component
try:
  from streamlit_autorefresh import st_autorefresh
except ImportError:
  st.error("Missing dependency: streamlit-autorefresh")

# Page Configuration
st.set_page_config(
    page_title="Nifty 500 Intraday Gainer Screener",
    page_icon="⚡",
    layout="wide",
)

# Auto-refresh every 10 seconds (Condition 3)
st_autorefresh(interval=10000, key="nifty500_screener_web")

st.title("⚡ Nifty 500 Intraday Gainer & Breakout Web Screener")
st.markdown(
    "Live web dashboard refreshing every **10 seconds**. Filters: Exclusions,"
    " Volume > 1M, Post-9:20 AM Volume Spikes."
)

# --- EXCLUSION RULES (Condition 1) ---
EXCLUDED_SECTORS_KEYWORDS = [
    "BANK",
    "OIL",
    "GAS",
    "PETRO",
    "STEEL",
    "POWER",
    "NTPC",
    "ONGC",
    "IOC",
    "BPCL",
    "HPCL",
    "GAIL",
    "COAL",
    "VEDL",
    "INFY",
    "TATA",
    "RELIANCE",
    "GOVT",
]
EXCLUDED_EXACT_TICKERS = [
    "INFY.NS",
    "VEDL.NS",
    "TATASTEEL.NS",
    "TATAMOTORS.NS",
    "TATAELXSI.NS",
    "TATACONSUM.NS",
    "TATAPOWER.NS",
    "TITAN.NS",
    "TRENT.NS",
    "VOLTAS.NS",
]


@st.cache_data(ttl=30)
def get_nifty_500_symbols():
  return [
      "RELIANCE.NS",
      "TCS.NS",
      "HDFCBANK.NS",
      "INFY.NS",
      "ICICIBANK.NS",
      "SBIN.NS",
      "BHARTIARTL.NS",
      "ITC.NS",
      "KOTAKBANK.NS",
      "LT.NS",
      "AXISBANK.NS",
      "HINDUNILVR.NS",
      "SUNPHARMA.NS",
      "BAJFINANCE.NS",
      "MARUTI.NS",
      "TITAN.NS",
      "TATAMOTORS.NS",
      "WIPRO.NS",
      "ULTRACEMCO.NS",
      "ASIANPAINT.NS",
      "NESTLEIND.NS",
      "NTPC.NS",
      "POWERGRID.NS",
      "M&M.NS",
      "TATASTEEL.NS",
      "BAJAJFINSV.NS",
      "GRASIM.NS",
      "HCLTECH.NS",
      "ADANIENT.NS",
      "ADANIPORTS.NS",
      "COALINDIA.NS",
      "BPCL.NS",
      "HINDALCO.NS",
      "ONGC.NS",
      "DIVISLAB.NS",
      "EICHERMOT.NS",
      "DRREDDY.NS",
      "SBILIFE.NS",
      "BAJAJ-AUTO.NS",
      "BRITANNIA.NS",
      "TATACONSUM.NS",
      "APOLLOHOSP.NS",
      "HEROMOTOCO.NS",
      "UPL.NS",
      "INDUSINDBNK.NS",
      "ZOMATO.NS",
      "NYKAA.NS",
      "PAYTM.NS",
      "POLICYBZR.NS",
      "DELHIVERY.NS",
      "PERSISTENT.NS",
      "COFORGE.NS",
      "LTIM.NS",
      "MPHASIS.NS",
      "LTTS.NS",
      "NAUKRI.NS",
      "POLYCAB.NS",
      "DIXON.NS",
      "HAL.NS",
      "BEL.NS",
      "IRCTC.NS",
      "RVNL.NS",
      "IDEA.NS",
      "IDFCFIRSTB.NS",
      "PNB.NS",
      "BANKBARODA.NS",
      "CANBK.NS",
      "VEDL.NS",
  ]


def should_exclude(ticker):
  ticker_upper = ticker.upper()
  if ticker_upper in EXCLUDED_EXACT_TICKERS:
    return True
  for kw in EXCLUDED_SECTORS_KEYWORDS:
    if kw in ticker_upper:
      return True
  return False


@st.cache_data(ttl=10)
def fetch_live_data():
  symbols = get_nifty_500_symbols()
  data_records = []

  try:
    tickers_str = " ".join(symbols[:30])
    data = yf.download(
        tickers_str, period="1d", interval="1m", group_by="ticker", progress=False
    )

    for symbol in symbols[:30]:
      try:
        df = data[symbol].dropna()
        if df.empty or len(df) < 5:
          continue

        current_price = float(df["Close"].iloc[-1])
        prev_close = float(df["Close"].iloc[0])
        pct_change = ((current_price - prev_close) / prev_close) * 100
        total_vol = int(df["Volume"].sum())

        # Condition 2: Volume > 1,000,000
        if total_vol < 1000000:
          continue

        # Condition 1: Exclude unwanted groups
        if should_exclude(symbol):
          continue

        # Condition 4b: Volume Spike Detector (After 9:20 AM)
        df_filtered_time = df.between_time("09:20", "15:30")
        has_volume_spike = False
        if not df_filtered_time.empty:
          mean_vol = df_filtered_time["Volume"].mean()
          max_vol_after_920 = df_filtered_time["Volume"].max()
          if max_vol_after_920 > (mean_vol * 3.0):
            has_volume_spike = True

        # Condition 4a: Breakout check
        day_high = float(df["High"].max())
        is_breakout = current_price >= (day_high * 0.995)

        data_records.append({
            "Symbol": symbol,
            "Price": round(current_price, 2),
            "Change (%)": round(pct_change, 2),
            "Volume": total_vol,
            "Volume Spike (>9:20)": "🔥 SPIKE" if has_volume_spike else "Normal",
            "Breakout": is_breakout,
        })
      except Exception:
        continue
  except Exception as e:
    st.error(f"Market fetch error: {e}")

  return pd.DataFrame(data_records)


# --- RENDER UI ---
df_live = fetch_live_data()

if df_live.empty:
  st.info(
      "🔄 Loading data / Markets might be closed right now. Live feed will"
      " display when active."
  )
else:
  df_gainers = df_live[df_live["Change (%)"] > 0].sort_values(
      by="Change (%)", ascending=False
  )

  # Sidebar Breakout Box (Condition 4a)
  st.sidebar.header("🚀 Breakout Stocks Box")
  breakout_stocks = df_gainers[df_gainers["Breakout"] == True]

  if not breakout_stocks.empty:
    for _, row in breakout_stocks.iterrows():
      st.sidebar.success(
          f"**{row['Symbol']}**\n"
          f"Price: ₹{row['Price']} | +{row['Change (%)']}%"
      )
  else:
    st.sidebar.info("No active breakouts right now.")

  # Main Table Display
  st.subheader("📈 Filtered Intraday Gainers")
  st.dataframe(
      df_gainers[
          [
              "Symbol",
              "Price",
              "Change (%)",
              "Volume",
              "Volume Spike (>9:20)",
          ]
      ],
      use_container_width=True,
  )
