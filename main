# main.py  ← Paste this EXACTLY (no secrets needed anymore!)
from pocketoptionapi_async import AsyncPocketOptionClient
import asyncio
import talib
import numpy as np
import telegram
from datetime import datetime
import pytz

# === YOUR INFO (ALREADY HARD-CODED) ===
SSID = "UiaMkFe9DN1cUd6OkI-luEW7SBLBUr8QW3ey9w"
TELEGRAM_TOKEN = "8316841867:AAFVxmQN3Ea-omWcCNtJuFn8uTxPHJKcaQo"
CHAT_ID = "5699045459"  # John (@mrgoty)

# === ZAMBIA TIME (CAT) ===
tz_cat = pytz.timezone('Africa/Lusaka')
current_time_cat = lambda: datetime.now(tz_cat).strftime('%I:%M %p CAT')

# === SETTINGS ===
ASSET = "EURUSD"
DURATION = 60
DEMO = True  # Unlimited demo money

bot = telegram.Bot(token=TELEGRAM_TOKEN)

async def send_signal(direction):
    msg = f"POCKET OPTION SIGNAL\n" \
          f"{ASSET} → {direction.upper()}\n" \
          f"Duration: {DURATION}s\n" \
          f"Time: {current_time_cat()}\n" \
          f"Open trade NOW! Zambia"
    await bot.send_message(chat_id=CHAT_ID, text=msg)

async def main():
    print(f"Starting Pocket Option Signal Bot — Zambia")
    print(f"Time: {current_time_cat()} | User: John (@mrgoty)")
    client = AsyncPocketOptionClient(SSID, is_demo=DEMO)
    await client.connect()
    print("Connected! Waiting for signals... Check Telegram @Kartelmoney_bot")

    checked = set()

    while True:
        try:
            candles = await client.get_candles(ASSET, 60, 100)
            if len(candles) < 50:
                await asyncio.sleep(5)
                continue

            closes = np.array([c.close for c in candles])
            rsi = talib.RSI(closes, timeperiod=14)[-1]
            ema_fast = talib.EMA(closes, timeperiod=9)[-1]
            ema_slow = talib.EMA(closes, timeperiod=21)[-1]
            prev_fast = talib.EMA(closes[:-1], timeperiod=9)[-1]
            prev_slow = talib.EMA(closes[:-1], timeperiod=21)[-1]

            current_time = candles[-1].timestamp
            minute_key = current_time // 60

            if minute_key in checked:
                await asyncio.sleep(3)
                continue

            # CALL SIGNAL
            if prev_fast <= prev_slow and ema_fast > ema_slow and rsi < 65:
                await send_signal("CALL")
                checked.add(minute_key)
                print(f"CALL sent at {current_time_cat()}")

            # PUT SIGNAL
            elif prev_fast >= prev_slow and ema_fast < ema_slow and rsi > 35:
                await send_signal("PUT")
                checked.add(minute_key)
                print(f"PUT sent at {current_time_cat()}")

            await asyncio.sleep(3)

        except Exception as e:
            print("Error (retrying in 10s):", e)
            await asyncio.sleep(10)

    await client.disconnect()

asyncio.run(main())
