import asyncio import requests import json from telegram import InlineKeyboardButton, InlineKeyboardMarkup, Update, Bot from telegram.ext import Application, CallbackQueryHandler, CommandHandler, ContextTypes

=== CONFIG ===

BOT_TOKEN = "7420147268:AAHnguB1Y0ZaJ6qF4U8-KqQ_t_gC6gtcuDQ" HELIUS_API_KEY = "b0b224fa-0850-4e15-8068-e48184260227" WALLET_ADDRESS = "EymZLSBXY23byVei45MimQ33FcSfNxNu6k6V8pPWNWEM"

=== FILTER DEFAULTS ===

filters = { "liquidity_min": 4000, "liquidity_max": 15000, "market_cost_min": 0.00000000200, "market_cost_max": 2, "dev_holding_min": 1, "dev_holding_max": 4, "pool_supply_min": 70, "pool_supply_max": 95, "slippage_min": 8, "slippage_max": 30, "auto_snipe": False, "bot_active": False }

=== UI BUTTONS ===

def main_menu(): buttons = [ [InlineKeyboardButton("🚀 Active" if filters["bot_active"] else "❌ Inactive", callback_data="toggle_active")], [InlineKeyboardButton("🔫 Auto Snipe: ON" if filters["auto_snipe"] else "🔫 Auto Snipe: OFF", callback_data="toggle_sniping")], [InlineKeyboardButton("🛠️ Filters", callback_data="view_filters")] ] return InlineKeyboardMarkup(buttons)

=== TELEGRAM COMMANDS ===

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE): await update.message.reply_text("Welcome to HANUMAN Sniper Bot", reply_markup=main_menu())

async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE): query = update.callback_query await query.answer()

if query.data == "toggle_active":
    filters["bot_active"] = not filters["bot_active"]
elif query.data == "toggle_sniping":
    filters["auto_snipe"] = not filters["auto_snipe"]
elif query.data == "view_filters":
    msg = "\n".join([f"{key}: {value}" for key, value in filters.items()])
    await query.edit_message_text(f"Current Filters:\n{msg}", reply_markup=main_menu())
    return

await query.edit_message_reply_markup(reply_markup=main_menu())

=== REAL-TIME HELIUS TRACKING ===

async def scan_new_tokens(bot: Bot): seen = set() while True: if filters["bot_active"]: url = f"https://mainnet.helius-rpc.com/?api-key={HELIUS_API_KEY}" headers = {'Content-Type': 'application/json'} payload = { "jsonrpc": "2.0", "id": 1, "method": "getSignaturesForAddress", "params": [WALLET_ADDRESS, {"limit": 10}] } try: res = requests.post(url, headers=headers, data=json.dumps(payload)).json() for tx in res.get("result", []): sig = tx.get("signature") if sig and sig not in seen: seen.add(sig) # Apply filter check logic here await bot.send_message(chat_id=WALLET_ADDRESS, text=f"New token tx: {sig}") except Exception as e: print("Scan Error:", e) await asyncio.sleep(5)

=== RUN ===

def main(): application = Application.builder().token(BOT_TOKEN).build() application.add_handler(CommandHandler("start", start)) application.add_handler(CallbackQueryHandler(button_handler))

bot = Bot(BOT_TOKEN)
asyncio.create_task(scan_new_tokens(bot))

application.run_polling()

if name == 'main': main()

