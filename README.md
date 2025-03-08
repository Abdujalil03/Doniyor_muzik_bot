import os
import yt_dlp
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, CallbackContext
from dotenv import load_dotenv

# Bot tokeni
TOKEN = "7903379258:AAEZfHOT-A3-XRvYOQZ1uXOiwaY4UO3aWbE"

# Qo‘shiqlar saqlanadigan papka
DOWNLOAD_PATH = "songs/"

if not os.path.exists(DOWNLOAD_PATH):
    os.makedirs(DOWNLOAD_PATH)

# /start komandasi uchun funksiya
async def start(update: Update, context: CallbackContext):
    await update.message.reply_text("🎵 Salom! Qo‘shiq yoki qo‘shiqchi nomini yozing, men 5 ta qo‘shiq topib beraman!")

# Qo‘shiq yuklab berish funksiyasi
async def download_song(update: Update, context: CallbackContext):
    query = update.message.text  # Foydalanuvchi yozgan matn
    await update.message.reply_text(f"🔍 '{query}' bo‘yicha qo‘shiqlarni qidiryapman...")

    ydl_opts = {
        'format': 'bestaudio/best',
        'outtmpl': f"{DOWNLOAD_PATH}/%(title)s.%(ext)s",
        'noplaylist': True,
        'postprocessors': [{
            'key': 'FFmpegExtractAudio',
            'preferredcodec': 'mp3',
            'preferredquality': '192',
        }],
    }

    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        try:
            info = ydl.extract_info(f"ytsearch5:{query}", download=True)

            for entry in info['entries']:
                song_title = entry['title']
                song_filename = f"{DOWNLOAD_PATH}/{song_title}.mp3"

                if os.path.exists(song_filename):
                    with open(song_filename, "rb") as song:
                        await update.message.reply_audio(song)
                    await update.message.reply_text(f"🎶 {song_title} yuklandi!")
                else:
                    await update.message.reply_text(f"⚠️ Xatolik! '{song_title}' yuklanmadi.")

        except Exception as e:
            await update.message.reply_text(f"❌ Xatolik yuz berdi: {str(e)}")

# Botni ishga tushirish
app = Application.builder().token(TOKEN).build()

app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, download_song))

print("✅ Bot ishga tushdi...")
app.run_polling()
