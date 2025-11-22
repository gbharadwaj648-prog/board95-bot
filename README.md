# board95-botimport os
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes
import google.generativeai as genai
from PIL import Image
import io
import pytesseract

# Your keys (paste here – get BotFather token from @BotFather, Gemini from aistudio.google.com/app/apikey)
TELEGRAM_TOKEN = "8562309961:AAH23j_Liuf-lE76YUTUISOF_9oDQrVdFjU"  # e.g., "7239456782:AAH..."
GEMINI_KEY = "AIzaSyAb3xECVwDwZHtbKf8I5P1IRr0B76jW5FM"  # e.g., "AIzaSyD..."

genai.configure(api_key=GEMINI_KEY)
model = genai.GenerativeModel("gemini-2.5-flash-exp")  # Latest model, no errors

# Simple in-memory counter for free doubts (resets on restart; use Redis for production)
user_doubts = {}

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    user_doubts[user_id] = user_doubts.get(user_id, 0)
    await update.message.reply_text(
        "🔥 95%+ Guarantee Bot – Class 10/12 Boards!\n\n"
        "Send doubt photo/text → Instant English solution.\n"
        "First 5 doubts FREE | Lifetime Unlimited + Tests = ₹499 only.\n\n"
        "Ask anything now! 📚"
    )

async def handle_doubt(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    user_doubts[user_id] = user_doubts.get(user_id, 0) + 1
    doubt_count = user_doubts[user_id]

    question = ""
    if update.message.photo:
        file = await update.message.photo[-1].get_file()
        bytes_data = await file.download_as_bytearray()
        img = Image.open(io.BytesIO(bytes_data))
        question = pytesseract.image_to_string(img, lang='eng')
    elif update.message.text:
        question = update.message.text

    if not question.strip():
        await update.message.reply_text("Photo unclear? Send a sharper one! 📸")
        return

    if doubt_count > 5:
        await update.message.reply_text(
            "Hello! You've used 5 free doubts.\n"
            "Get Unlimited + Daily Tests + 95% Guarantee for ₹499 (one-time).\n\n"
            "UPI: bharadwaj648@oksbi\n"
            "Send payment screenshot → Unlocked forever!\n"
            "Money back if below 95% 😊"
        )
        return

    await update.message.reply_text("Solving... (10 sec) ⏳")

    prompt = f"""
    You are a top state board tutor (95%+ scorer). Reply in pure English, step-by-step.

    Question: {question}

    Format:
    1. Steps with numbers
    2. Short trick for full marks
    3. Diagram description if needed
    4. End with: "More doubts? Lifetime only ₹499 → UPI: bharadwaj648@oksbi"

    Keep under 500 words.
    """
    try:
        response = model.generate_content(prompt)
        await update.message.reply_text(response.text)
    except Exception as e:
        await update.message.reply_text(f"Oops! Try again. (Error: {e})")

def main():
    app = Application.builder().token(TELEGRAM_TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.PHOTO | filters.TEXT & ~filters.COMMAND, handle_doubt))
    print("Board95 Bot is LIVE!")
    app.run_polling(drop_pending_updates=True)

if __name__ == '__main__':
    main()