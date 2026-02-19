import telebot
import os
import csv
import random
from datetime import datetime
from telebot.types import ReplyKeyboardMarkup
from flask import Flask, request

# -------------------------------
# TOKEN (en Secrets de Replit)

TOKEN = os.environ.get("PIXELINA_TOKEN")
if not TOKEN:
    raise ValueError("PIXELINA_TOKEN no definido en Secrets")

bot = telebot.TeleBot(TOKEN)
app = Flask(__name__)

# 🔐 TU ID
ADMIN_ID = 1551887836

# -------------------------------
# CONFIG WEBHOOK PARA REPLIT

REPL_URL = os.environ.get("REPLIT_DEV_DOMAIN")

if not REPL_URL:
    raise ValueError("REPLIT_DEV_DOMAIN no definido")

WEBHOOK_URL = f"https://{REPL_URL}/{TOKEN}"

bot.remove_webhook()
bot.set_webhook(url=WEBHOOK_URL)

print("✅ Webhook configurado en Replit")

# -------------------------------
# TEXTOS

wifi_info = "📶 Red: Estudiantes\n🔑 Contraseña: Escuelas_2025"

tareas_msgs = [
    "📚 ¡Hacelas! No dejes para último momento.",
    "📝 Revisá Classroom."
]

profe_msgs = [
    "👩‍🏫 Consultá su horario institucional.",
    "📧 Podés escribirle por mail."
]

oraculo_msgs = [
    "🔮 Hoy será un gran día.",
    "✨ Confía en tu intuición."
]

# -------------------------------
# MENÚ

def main_menu():
    markup = ReplyKeyboardMarkup(resize_keyboard=True)
    markup.row("📶 Wifi", "📚 Tareas")
    markup.row("👩‍🏫 Profe", "🔮 Oráculo")
    markup.row("💡 Sugerencia", "🆘 Ayuda")
    markup.row("🏠 Inicio")
    return markup

# -------------------------------
# GUARDAR CSV

def guardar_registro(archivo, datos):
    existe = os.path.isfile(archivo)

    with open(archivo, "a", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)

        if not existe:
            writer.writerow(["usuario", "mensaje", "fecha"])

        writer.writerow(datos)

# -------------------------------
# START

@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(
        message.chat.id,
        "👋 Hola, soy PixelinaBot 🤖",
        reply_markup=main_menu()
    )

# -------------------------------
# HANDLER GENERAL

@bot.message_handler(func=lambda m: True)
def responder(message):

    if not message.text:
        return

    txt = message.text.lower()

    if "wifi" in txt:
        bot.send_message(message.chat.id, wifi_info)

    elif "tareas" in txt:
        bot.send_message(message.chat.id, random.choice(tareas_msgs))

    elif "profe" in txt:
        bot.send_message(message.chat.id, random.choice(profe_msgs))

    elif "oráculo" in txt or "oraculo" in txt:
        bot.send_message(message.chat.id, random.choice(oraculo_msgs))

    elif "sugerencia" in txt:
        msg = bot.send_message(message.chat.id, "✍️ Escribí tu sugerencia.")
        bot.register_next_step_handler(msg, guardar_sugerencia)

    elif "ayuda" in txt:
        msg = bot.send_message(message.chat.id, "📨 Escribí tu consulta.")
        bot.register_next_step_handler(msg, guardar_ayuda)

    elif "inicio" in txt:
        bot.send_message(message.chat.id, "Menú principal 👇", reply_markup=main_menu())

    else:
        bot.send_message(message.chat.id, "No entendí 🤖", reply_markup=main_menu())

# -------------------------------
# FUNCIONES

def guardar_sugerencia(message):
    fecha = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    usuario = message.from_user.first_name
    guardar_registro("sugerencias.csv", [usuario, message.text, fecha])

    bot.send_message(
        ADMIN_ID,
        f"📩 NUEVA SUGERENCIA\n\n👤 {usuario}\n📝 {message.text}\n📅 {fecha}"
    )

    bot.send_message(message.chat.id, "✅ Guardada.", reply_markup=main_menu())

def guardar_ayuda(message):
    fecha = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    usuario = message.from_user.first_name
    guardar_registro("ayuda.csv", [usuario, message.text, fecha])

    bot.send_message(
        ADMIN_ID,
        f"🆘 NUEVA CONSULTA\n\n👤 {usuario}\n📝 {message.text}\n📅 {fecha}"
    )

    bot.send_message(message.chat.id, "✅ Registrada.", reply_markup=main_menu())

# -------------------------------
# WEBHOOK

@app.route(f"/{TOKEN}", methods=["POST"])
def webhook():
    json_str = request.get_data().decode("UTF-8")
    update = telebot.types.Update.de_json(json_str)
    bot.process_new_updates([update])
    return "OK", 200

# -------------------------------
# RUN

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
