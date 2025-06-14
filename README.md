import logging
import requests
from bs4 import BeautifulSoup
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

# Token de tu bot
TOKEN = "AAFUB9SesajLQLTrq-vUogMkzq2AC88W_3s"

logging.basicConfig(level=logging.INFO)

async def check(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        await update.message.reply_text("❌ Usa el comando así:\n/check 353915101XXXXXX o SN123456")
        return

    code = context.args[0].strip()
    await update.message.reply_text("🔍 Consultando información...")

    result = ""

    # --- Información técnica desde sndeep.info ---
    try:
        info = requests.get(f"https://sndeep.info/en?q={code}")
        soup = BeautifulSoup(info.text, 'html.parser')
        data_div = soup.find('div', class_="block_result")
        if data_div:
            result += "📱 *Especificaciones del dispositivo:*\n"
            result += data_div.get_text(separator="\n").strip() + "\n\n"
        else:
            result += "⚠️ No se encontró información técnica.\n\n"
    except Exception as e:
        result += f"⚠️ Error al consultar especificaciones: {str(e)}\n\n"

    # --- Verificación en iRemove.tools ---
    try:
        ir_response = requests.post(
            "https://iremove.tools/es/check-order-status",
            data={"sn": code}
        )
        soup = BeautifulSoup(ir_response.text, 'html.parser')
        result_div = soup.find("div", class_="check-imei-result")
        if result_div:
            result += "🔐 *Bypass (iRemove.tools):*\n"
            result += result_div.get_text(separator="\n").strip() + "\n\n"
        else:
            result += "⚠️ No se encontró estado de bypass en iRemove.\n\n"
    except Exception as e:
        result += f"⚠️ Error iRemove: {str(e)}\n\n"

    # --- Verificación en iRemovalPro.com ---
    try:
        irp_response = requests.post(
            "https://iremovalpro.com/Paya12.php",
            data={"search": code}
        )
        soup = BeautifulSoup(irp_response.text, 'html.parser')
        table = soup.find("table")
        if table:
            result += "🔐 *Bypass (iRemoval Pro):*\n"
            result += table.get_text(separator="\n").strip()
        else:
            result += "⚠️ No se encontró estado de bypass en iRemoval Pro."
    except Exception as e:
        result += f"⚠️ Error iRemoval Pro: {str(e)}"

    await update.message.reply_text(result, parse_mode="Markdown")

if __name__ == "__main__":
    app = ApplicationBuilder().token(TOKEN).build()
    app.add_handler(CommandHandler("check", check))
    app.run_polling()
