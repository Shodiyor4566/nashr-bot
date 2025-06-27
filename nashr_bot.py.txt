from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, ContextTypes, filters

# Bot token va admin ID
TOKEN = "7828933833:AAEgCXThirc1a8qBK88lYGDYVsL4tKb3Qa8"
ADMIN_ID = 7477414895

# Foydalanuvchi bosqichlari uchun joy
user_steps = {}

# /start komandasi
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.chat_id
    user_steps[user_id] = {"step": 1}
    await update.message.reply_text("👋 Assalomu alaykum! Buyurtma berish uchun ismingizni yozing:")

# Har qanday yozilgan xabarlarni boshqarish
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.chat_id
    text = update.message.text

    if user_id not in user_steps:
        await update.message.reply_text("Iltimos /start buyrug‘ini bosing.")
        return

    step = user_steps[user_id]["step"]

    if step == 1:
        user_steps[user_id]["name"] = text
        user_steps[user_id]["step"] = 2
        await update.message.reply_text("📞 Endi telefon raqamingizni yozing:")
    elif step == 2:
        user_steps[user_id]["phone"] = text
        user_steps[user_id]["step"] = 3
        await update.message.reply_text("📚 Qaysi kitobni xohlaysiz?")
    elif step == 3:
        user_steps[user_id]["book"] = text
        user_steps[user_id]["step"] = 4
        await update.message.reply_text("📍 Yetkazib berish manzilingizni yozing:")
    elif step == 4:
        user_steps[user_id]["address"] = text

        # Yig‘ilgan ma'lumotlar
        name = user_steps[user_id]["name"]
        phone = user_steps[user_id]["phone"]
        book = user_steps[user_id]["book"]
        address = user_steps[user_id]["address"]

        # Adminga xabar
        summary = (
            f"✅ Yangi buyurtma:\n"
            f"👤 Ism: {name}\n"
            f"📞 Tel: {phone}\n"
            f"📚 Kitob: {book}\n"
            f"📍 Manzil: {address}"
        )

        await update.message.reply_text("✅ Buyurtmangiz qabul qilindi! Tez orada aloqaga chiqamiz.")
        await context.bot.send_message(chat_id=ADMIN_ID, text=summary)

        # Tozalash
        del user_steps[user_id]

# Botni ishga tushirish
app = ApplicationBuilder().token(TOKEN).build()
app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

print("✅ Bot ishga tushdi...")
app.run_polling()
