# Design-project-2-
Design project 2.
from alpha_vantage.timeseries import TimeSeries
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import random
import numpy as np
from tabulate import tabulate
import json
import os
import webbrowser

PROFILE_FILE = "user_profile.json"

def load_profile(reset=False):
    """Loads the user profile from a JSON file or initializes a new profile."""
    if reset or not os.path.exists(PROFILE_FILE):
        profile = {"points": 0, "badges": [], "investments": []}
        save_profile(profile)
        return profile
    with open(PROFILE_FILE, "r") as file:
        return json.load(file)

def save_profile(profile):
    """Saves the user profile to a JSON file."""
    with open(PROFILE_FILE, "w") as file:
        json.dump(profile, file, indent=4)

def award_points(profile, action):
    """Awards points based on user actions."""
    points_map = {
        "run_analysis": 10,
        "watch_videos": 5,
        "view_guide": 5
    }
    earned = points_map.get(action, 0)
    if earned > 0:
        profile["points"] += earned
        print(f"\n🏆 You earned {earned} points! Total: {profile['points']}")
    return profile

def check_badges(profile):
    """Awards badges based on points."""
    badge_conditions = [
        (10, "🥇 First Step"),
        (50, "🌟 Rising Star"),
        (100, "🔮 Market Wizard"),
    ]
    for points_needed, badge in badge_conditions:
        if profile["points"] >= points_needed and badge not in profile["badges"]:
            profile["badges"].append(badge)
            print(f"🏅 Badge Unlocked: {badge}!")
    return profile

def show_profile(profile):
    """Displays profile details."""
    print("\n📌 User Profile:")
    print(f"🏆 Total Points: {profile.get('points', 0)}")
    badges = profile.get("badges", [])
    if badges:
        print(f"🎖️ Badges: {', '.join(badges)}")
    else:
        print("🎖️ Badges: None")
    investments = profile.get("investments", [])
    if investments:
        print("💰 Investments:")
        for stock in investments:
            print(f"  - {stock['symbol']} | ${stock['amount']} | {stock['status']}")
    else:
        print("💰 No investments made yet.")
    print("=" * 40)

def beginner_investment_guide():
    guide = """
    Beginner-Friendly Investment Guide
    -----------------------------------
     1. Understanding Stock Market Basics:
       - Stocks represent ownership in a company.
       - Prices fluctuate based on supply, demand, and market conditions.
       - Common order types:
         * Market Order: Buy/sell immediately at the current price.
         * Limit Order: Buy/sell at a specific price or better.
         * Stop-Loss Order: Automatically sell if the price drops to a set level.

    2. Key Stock Metrics Explained:
       - *P/E Ratio (Price-to-Earnings)*: Measures valuation; lower is usually better.
       - *EPS (Earnings Per Share)*: Shows company profitability.
       - *Market Cap*: Total market value of a company's shares.
       - *Dividend Yield*: Percentage return from dividends.
       - *RSI (Relative Strength Index)*: Measures stock momentum (above 70 = overbought, below 30 = oversold).
       
    3. Risk Assessment for New Investors:
       - *Low Risk*: Blue-chip stocks, index funds (e.g., S&P 500 ETFs).
       - *Medium Risk*: Large companies with strong growth (e.g., tech stocks, healthcare).
       - *High Risk*: Small-cap stocks, crypto, speculative investments.

    4. Investment Strategy Tips:
       - Diversify your portfolio to reduce risk.
       - Start with long-term investments.
       - Avoid emotional trading—stick to your strategy.
       - Use stop-loss orders to protect investments.

    5. Suggested Beginner-Friendly Stocks:

       - *Tech Giants*: AAPL (Apple), MSFT (Microsoft), GOOG (Google).
       - *Dividend Stocks*: KO (Coca-Cola), JNJ (Johnson & Johnson), PEP (PepsiCo).
       - *ETFs for Diversification*: SPY (S&P 500 ETF), VTI (Total Stock Market ETF).
    """
    print(guide)

def beginner_friendly_videos():
    videos = {
        "1": ("Stock Market Basics", "https://youtu.be/oHAB6f-P-K0?si=fMLMQuWJUA6nF_n5"),
        "2": ("How to Read Stock Charts", "https://youtu.be/J-ntsk7Dsd0?si=PhxMBibJFU3bhzp5"),
        "3": ("Investing Strategies for Beginners", "https://youtu.be/Jd8Xv1v51vU?si=DvxxWC-ynTIi1Ewb"),
        "4": ("Technical Analysis Basics", "https://youtu.be/GmsCvpYE50c?si=ApBwsc8PBXxUl4eX")
    }

    print("\n📺 Recommended Investment Videos for Beginners:\n")
    for num, (title, _) in videos.items():
        print(f"{num}. {title}")

    choices = input("\nEnter the number(s) of the video(s) you want to watch (comma-separated): ").strip()
    if choices:
        profile = load_profile()
        for choice in choices.split(","):
            choice = choice.strip()
            if choice in videos:
                webbrowser.open(videos[choice][1])
                print(f"Opening: {videos[choice][0]}...")
        profile = award_points(profile, "watch_videos")
        profile = check_badges(profile)
        save_profile(profile)
    else:
        print("No videos selected.")

API_KEY = "your_alpha_vantage_api_key"

def get_stock_data_alpha_vantage(ticker):
    ts = TimeSeries(key=API_KEY, output_format='pandas')
    try:
        data, _ = ts.get_daily(symbol=ticker, outputsize='compact')
        return data['4. close'].iloc[:14]
    except Exception as e:
        return f"Error fetching data: {e}"

def get_market_trend_and_prediction(ticker):
    recent_prices = get_stock_data_alpha_vantage(ticker)
    if isinstance(recent_prices, str): return (recent_prices, None, None, None)

    recent_prices = recent_prices.iloc[::-1]
    price_change = recent_prices.pct_change().dropna()
    avg_change = price_change.mean()
    predicted_change_percentage = avg_change * 100
    avg_daily_change = np.abs(price_change).mean() * 100
    avg_days = abs(predicted_change_percentage) / avg_daily_change if avg_daily_change > 0 else None

    if avg_change > 0.005:
        return ("Bullish", predicted_change_percentage, recent_prices, avg_days)
    elif avg_change < -0.005:
        return ("Bearish", predicted_change_percentage, recent_prices, avg_days)
    else:
        return ("Sideways", predicted_change_percentage, recent_prices, avg_days)

def calculate_investment_amount(balance, trend, risk):
    risk_map = {
        "Bullish": {"high": 0.6, "low": 0.3},
        "Sideways": {"high": 0.3, "low": 0.15},
        "Bearish": {"high": 0.15, "low": 0.05}
    }
    return balance * risk_map[trend][risk]

def suggest_alternative_stocks(trend):
    if trend == "Bullish":
        return random.sample(["NVDA", "TSLA", "AMZN", "MSFT", "AAPL"], 3)
    elif trend == "Bearish":
        return random.sample(["JNJ", "PG", "KO", "PEP", "WMT"], 3)
    else:
        return random.sample(["SPY", "VTI", "QQQ", "DIA", "ARKK"], 3)

def plot_stock_trend(ticker, prices):
    plt.figure(figsize=(10, 5))
    plt.plot(prices.index, prices.values, marker='o', linestyle='-', label=ticker)
    plt.xlabel("Date")
    plt.ylabel("Closing Price")
    plt.title(f"{ticker} Stock Trend (14 Days)")
    plt.xticks(rotation=45)
    plt.grid()
    plt.legend()
    plt.tight_layout()
    plt.show()

def calculate_rsi(prices, period=14):
    if len(prices) < period: return None
    delta = prices.diff()
    gain = (delta.where(delta > 0, 0)).rolling(period).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(period).mean()
    rs = gain / loss
    return 100 - (100 / (1 + rs))

def analyze_multiple_stocks(tickers, risk, balance):
    results = []
    plt.figure(figsize=(12, 6))
    for ticker in tickers:
        trend, pred_pct, prices, avg_days = get_market_trend_and_prediction(ticker)
        if isinstance(prices, str) or prices is None:
            results.append([ticker, "Error", "N/A", "N/A", "N/A", "N/A", "N/A"])
            continue
        rsi = calculate_rsi(prices)
        rsi_val = rsi.iloc[-1] if rsi is not None else "N/A"
        invest = calculate_investment_amount(balance, trend, risk)
        profit = (invest * pred_pct) / 100
        profit_str = f"${abs(profit):.2f} ({'Profit' if profit > 0 else 'Loss'})"
        alt_stocks = ", ".join(suggest_alternative_stocks(trend))
        results.append([ticker, trend, f"{pred_pct:+.2f}%", rsi_val, f"${invest:.2f}", profit_str, alt_stocks])
        plt.plot(prices.index, prices.values, label=ticker)

    plt.title("Stock Trend Comparison")
    plt.legend()
    plt.grid()
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()

    headers = ["Stock", "Trend", "Predicted %", "RSI", "Investment ($)", "Profit/Loss", "Alt Stocks"]
    return tabulate(results, headers=headers, tablefmt="grid")

# -------------------------------
# 🚀 Main Interactive Flow
# -------------------------------
profile = load_profile(reset=False)  # Set reset=True to clear all data

# Optionally show the guide
beginner_investment_guide()
profile = award_points(profile, "view_guide")
profile = check_badges(profile)
save_profile(profile)

# Optionally watch videos
beginner_friendly_videos()

# Get user input for stock analysis
tickers = input("Enter stock tickers (comma-separated): ").strip().upper().split(",")
risk_tolerance = input("Enter risk tolerance (high/low): ").strip().lower()
balance = float(input("Enter your investment balance: "))

# Perform analysis
report = analyze_multiple_stocks(tickers, risk_tolerance, balance)
print(report)

# Award points ansd update profile
profile = award_points(profile, "run_analysis")
profile = check_badges(profile)
save_profile(profile)
show_profile(profile)
