import os
import threading
from flask import Flask
import telebot
from telebot.types import ChatPermissions

BOT_NAME = "TomHelperBoT"
BOT_TOKEN = os.environ.get("BOT_TOKEN", "9126786606:AAFSpEojiYwDDWHFKexVMKu4pTWp5HIDac0")

ADMIN_IDS_STR = os.environ.get("ADMIN_IDS", "8635066797")
ADMIN_IDS = set(int(x.strip()) for x in ADMIN_IDS_STR.split(",") if x.strip().isdigit())

# Scammer ID များကို Environment Variable မှ ကြိုထည့်နိုင်သည် (ဥပမာ: "111111,222222")
SCAMMER_IDS_STR = os.environ.get("SCAMMER_IDS", "")
SCAMMER_IDS = set(int(x.strip()) for x in SCAMMER_IDS_STR.split(",") if x.strip().isdigit())

bot = telebot.TeleBot(BOT_TOKEN)
app = Flask(__name__)

# Default စိန်ဈေး / Pass ဈေးနှုန်းစာရင်း
CURRENT_PRICE_LIST = """💎 **MLBB DIAMOND & PASS PRICE LIST** 💎
━━━━━━━━━━━━━━━━━━
• Weekly Pass x1 : --- MMK
• 86 💎 : --- MMK
• 172 💎 : --- MMK
• 257 💎 : --- MMK
• 706 💎 : --- MMK
━━━━━━━━━━━━━━━━━━
📌 ဝယ်ယူလိုပါက Owner ဆီ ဆက်သွယ်ပါ။"""

# --- UptimeRobot အတွက် Web Server ---
@app.route('/')
def home():
    return f"{BOT_NAME} is alive and running!"

def run_web():
    port = int(os.environ.get("PORT", 8080))
    app.run(host="0.0.0.0", port=port)

# --- Helper ---
def is_admin(user_id):
    return user_id in ADMIN_IDS

def format_user_name(user):
    # Username ရှိရင် @username ပြမယ်၊ မပါရင် နာမည် (First/Last name) ပြမယ်
    if user.username:
        return f"@{user.username}"
    if user.first_name and user.last_name:
        return f"{user.first_name} {user.last_name}"
    return user.first_name or str(user.id)

# --- Commands ---
@bot.message_handler(commands=['start'])
def send_welcome(message):
    bot.reply_to(
        message,
        f"💎 မင်္ဂလာပါ! ကျွန်တော်က **{BOT_NAME}** ပါ။\n\n"
        "📌 **Commands များ:**\n"
        "• `close` / `/close` — Group ပိတ်ပြီး ဆိုင်ပိတ်ပါပြီ ဟုပြောရန် (Admin)\n"
        "• `open` / `/open` — Group ဖွင့်ပြီး ဆိုင်ဖွင့်ပါပြီ ဟုပြောရန် (Admin)\n"
        "• `/price` — လက်ရှိ ဈေးနှုန်းစာရင်း ကြည့်ရန်\n"
        "• `/setprice <စာသား>` — ဈေးနှုန်းစာရင်း ပြင်ရန် (Admin)\n"
        "• `/addscam <user_id>` — Scammer ID ထည့်ရန် (Admin)\n"
        "• `/delscam <user_id>` — Scammer ID ဖျက်ရန် (Admin)\n"
        "• `/scamlist` — Scammer ID စာရင်း ကြည့်ရန် (Admin)\n"
        "• `/id` — ကိုယ့် ID ကြည့်ရန်",
        parse_mode="Markdown"
    )

@bot.message_handler(commands=['id'])
@bot.message_handler(func=lambda m: m.text and m.text.strip().lower() == "id")
def show_id(message):
    bot.reply_to(
        message,
        f"🆔 **User ID:** `{message.from_user.id}`\n💬 **Chat ID:** `{message.chat.id}`",
        parse_mode="Markdown"
    )

@bot.message_handler(commands=['price'])
@bot.message_handler(func=lambda m: m.text and m.text.strip().lower() in ["price", "ဈေးနှုန်း", "စိန်ဈေး"])
def show_price(message):
    bot.send_message(message.chat.id, CURRENT_PRICE_LIST, parse_mode="Markdown")

@bot.message_handler(commands=['setprice'])
def set_price(message):
    global CURRENT_PRICE_LIST
    if not is_admin(message.from_user.id):
        return
    parts = message.text.split(maxsplit=1)
    if len(parts) < 2 or not parts[1].strip():
        bot.reply_to(
            message,
            "⚠️ ဈေးနှုန်းအသစ်ကို `/setprice` နောက်မှာ ရိုက်ထည့်ပေးပါ။\n\n"
            "ဥပမာ -\n`/setprice 💎 စိန်ဈေးသစ် 💎\n• 86 💎 - 5,000 MMK`",
            parse_mode="Markdown"
        )
        return
    CURRENT_PRICE_LIST = parts[1].strip()
    bot.reply_to(message, "✅ ဈေးနှုန်းစာရင်းကို အောင်မြင်စွာ ပြင်ဆင်ပြီးပါပြီ။")

# --- Scammer Management Commands ---
@bot.message_handler(commands=['addscam'])
def add_scammer(message):
    if not is_admin(message.from_user.id):
        return
    parts = message.text.split()
    if len(parts) < 2 or not parts[1].strip().isdigit():
        bot.reply_to(message, "⚠️ အသုံးပြုပုံ: `/addscam 123456789`", parse_mode="Markdown")
        return
    scam_id = int(parts[1].strip())
    SCAMMER_IDS.add(scam_id)
    bot.reply_to(message, f"🚨 ID `{scam_id}` ကို Scammer list ထဲ ထည့်သွင်းပြီးပါပြီ။", parse_mode="Markdown")

@bot.message_handler(commands=['delscam'])
def del_scammer(message):
    if not is_admin(message.from_user.id):
        return
    parts = message.text.split()
    if len(parts) < 2 or not parts[1].strip().isdigit():
        bot.reply_to(message, "⚠️ အသုံးပြုပုံ: `/delscam 123456789`", parse_mode="Markdown")
        return
    scam_id = int(parts[1].strip())
    if scam_id in SCAMMER_IDS:
        SCAMMER_IDS.remove(scam_id)
        bot.reply_to(message, f"✅ ID `{scam_id}` ကို Scammer list ထဲမှ ဖယ်ရှားပြီးပါပြီ။", parse_mode="Markdown")
    else:
        bot.reply_to(message, f"ℹ️ ID `{scam_id}` သည် Scammer list ထဲတွင် မရှိပါ။", parse_mode="Markdown")

@bot.message_handler(commands=['scamlist'])
def list_scammers(message):
    if not is_admin(message.from_user.id):
        return
    if not SCAMMER_IDS:
        bot.reply_to(message, "📋 Scammer list ထဲတွင် ID မရှိသေးပါ။")
        return
    formatted_ids = "\n".join(f"• `{uid}`" for uid in sorted(SCAMMER_IDS))
    bot.reply_to(message, f"🚨 **Scammer List ({len(SCAMMER_IDS)})**\n{formatted_ids}", parse_mode="Markdown")

# --- New Member Scammer Alert ---
@bot.message_handler(content_types=['new_chat_members'])
def check_new_members(message):
    for member in message.new_chat_members:
        if member.id in SCAMMER_IDS:
            user_name = format_user_name(member)
            bot.send_message(
                message.chat.id,
                f"⚠️ ယခုဝင်လာသော {user_name} သည် scammer list ထဲတွင်ရှိနေပါသည်"
            )

# --- Close / Open Shop in Group ---
@bot.message_handler(func=lambda m: m.text and m.text.strip().lower() in ["close", "/close"])
def close_shop(message):
    if message.chat.type == "private" or not is_admin(message.from_user.id):
        return
    try:
        permissions = ChatPermissions(can_send_messages=False)
        bot.set_chat_permissions(message.chat.id, permissions)
        bot.send_message(message.chat.id, "🔒 ဆိုင်ပိတ်ပါပြီ။")
    except Exception as e:
        bot.send_message(message.chat.id, f"⚠️ ပိတ်လို့မရပါ (Bot ကို Admin ပေးထား/မထား စစ်ပါ): {e}")

@bot.message_handler(func=lambda m: m.text and m.text.strip().lower() in ["open", "/open"])
def open_shop(message):
    if message.chat.type == "private" or not is_admin(message.from_user.id):
        return
    try:
        permissions = ChatPermissions(
            can_send_messages=True,
            can_send_media_messages=True,
            can_send_other_messages=True,
            can_add_web_page_previews=True
        )
        bot.set_chat_permissions(message.chat.id, permissions)
        bot.send_message(message.chat.id, "🔓 ဆိုင်ဖွင့်ပါပြီ။ စိန်ဈေးကို Pin မှာကြည့်နိုင်ပါတယ်။")
    except Exception as e:
        bot.send_message(message.chat.id, f"⚠️ ဖွင့်လို့မရပါ: {e}")

if __name__ == "__main__":
    threading.Thread(target=run_web, daemon=True).start()
    print(f"Starting {BOT_NAME}...")
    bot.infinity_polling(skip_pending=True)
