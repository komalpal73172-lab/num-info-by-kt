import telebot
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

BOT_TOKEN = "7893073268:AAGeaTuDKZ60mN_HE2wKJ0KBfc2K03aC17o"
API_BASE_URL = "https://travelers-creature-sarah-rogers.trycloudflare.com/search"

bot = telebot.TeleBot(BOT_TOKEN, threaded=True, num_threads=8)

session = requests.Session()
adapter = HTTPAdapter(pool_connections=20, pool_maxsize=20, max_retries=Retry(total=2, backoff_factor=0.2))
session.mount('https://', adapter)
session.mount('http://', adapter)

def get_emoji(key_name):
    key = key_name.lower()
    if any(w in key for w in ["name", "user"]):
        return "👤"
    elif any(w in key for w in ["phone", "mobile", "number", "contact"]):
        return "📞"
    elif any(w in key for w in ["id", "uid", "aadhaar", "cnic"]):
        return "🆔"
    elif any(w in key for w in ["address", "city", "state", "location", "country"]):
        return "📍"
    elif any(w in key for w in ["mail", "email"]):
        return "📧"
    elif any(w in key for w in ["date", "time", "dob", "created"]):
        return "📅"
    elif any(w in key for w in ["gender", "sex"]):
        return "⚧"
    elif any(w in key for w in ["status", "type"]):
        return "⚡"
    elif any(w in key for w in ["father", "guardian"]):
        return "👨‍👦"
    return "🔹"

def format_data(data, seen=None):
    if seen is None:
        seen = set()
    lines = []
    
    if isinstance(data, dict):
        for k, v in data.items():
            if isinstance(v, (dict, list)):
                nested = format_data(v, seen)
                if nested:
                    lines.append(f"📁 <b>{k.upper()}:</b>\n{nested}")
            else:
                val_str = str(v).strip()
                pair_key = f"{k}:{val_str}"
                if pair_key not in seen and val_str:
                    seen.add(pair_key)
                    icon = get_emoji(k)
                    lines.append(f"{icon} <b>{k.upper()} :</b> <b>{val_str}</b>")
    elif isinstance(data, list):
        for item in data:
            if isinstance(item, (dict, list)):
                nested = format_data(item, seen)
                if nested:
                    lines.append(nested)
                    lines.append("─────────────────────")
            else:
                val_str = str(item).strip()
                if val_str not in seen and val_str:
                    seen.add(val_str)
                    lines.append(f"👉 <b>{val_str}</b>")
    else:
        return f"<b>{str(data)}</b>"
        
    return "\n".join(lines)

def send_large_message(chat_id, text):
    chunk_size = 4000
    if len(text) <= chunk_size:
        bot.send_message(chat_id, text, parse_mode="HTML")
        return
    for i in range(0, len(text), chunk_size):
        bot.send_message(chat_id, text[i:i + chunk_size], parse_mode="HTML")

@bot.message_handler(commands=['start', 'help'])
def send_welcome(message):
    bot.reply_to(message, "👋 <b>Welcome!</b>\n\n🔍 <i>Koi bhi query bhejein search karne ke liye.</i>", parse_mode="HTML")

@bot.message_handler(func=lambda message: True)
def handle_query(message):
    query = message.text.strip()

    try:
        response = session.get(API_BASE_URL, params={"q": query}, timeout=10)
        if response.status_code == 200:
            try:
                json_data = response.json()
                formatted_text = format_data(json_data)
                if not formatted_text:
                    formatted_text = "❌ <b>No Data Found!</b>"
                else:
                    formatted_text = f"🎯 <b>SEARCH RESULTS:</b>\n\n{formatted_text}"
                send_large_message(message.chat.id, formatted_text)
            except Exception:
                send_large_message(message.chat.id, f"📋 <b>Data:</b>\n\n{response.text}")
        else:
            bot.send_message(message.chat.id, f"⚠️ <b>Error: Status {response.status_code}</b>", parse_mode="HTML")
    except Exception as e:
        bot.send_message(message.chat.id, f"🚨 <b>Request Error:</b> <code>{str(e)}</code>", parse_mode="HTML")

if __name__ == "__main__":
    print("🚀 Bot is running without build errors!")
    bot.infinity_polling(skip_pending=True)
