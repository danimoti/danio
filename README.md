# danio
from telegram import Update
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext
from pypdf import PdfReader, PdfWriter
import os

# توکن ربات خود را در اینجا قرار دهید
BOT_TOKEN = 7823241035:AAHR-k29E-y1H_1X0IEEwvY6ZhqPaXVvoD8

# پوشه موقت برای ذخیره فایل‌ها
TEMP_FOLDER = 'temp_files'
os.makedirs(TEMP_FOLDER, exist_ok=True)

# لیست فایل‌های دریافتی
received_files = []

def start(update: Update, context: CallbackContext) -> None:
    update.message.reply_text('سلام! لطفاً فایل‌های خود را ارسال کنید.')

def handle_file(update: Update, context: CallbackContext) -> None:
    file = update.message.document or update.message.photo[-1]
    file_id = file.file_id
    file_name = file.file_name if update.message.document else f"{file_id}.jpg"
    file_path = os.path.join(TEMP_FOLDER, file_name)

    # دانلود فایل
    file = context.bot.get_file(file_id)
    file.download(file_path)

    received_files.append(file_path)
    update.message.reply_text(f'فایل {file_name} دریافت شد.')

def merge_files(update: Update, context: CallbackContext) -> None:
    if not received_files:
        update.message.reply_text('هیچ فایلی برای ادغام وجود ندارد.')
        return

    output_pdf_path = os.path.join(TEMP_FOLDER, 'merged_output.pdf')
    pdf_writer = PdfWriter()

    for file_path in received_files:
        if file_path.endswith('.pdf'):
            pdf_reader = PdfReader(file_path)
            for page_num in range(len(pdf_reader.pages)):
                pdf_writer.add_page(pdf_reader.pages[page_num])
        else:
            # تبدیل تصاویر به PDF و اضافه کردن به فایل نهایی
            # این بخش نیاز به پیاده‌سازی دارد
            pass

    with open(output_pdf_path, 'wb') as out_pdf:
        pdf_writer.write(out_pdf)

    with open(output_pdf_path, 'rb') as out_pdf:
        update.message.reply_document(out_pdf, filename='merged_output.pdf')

    # پاکسازی فایل‌های موقت
    for file_path in received_files:
        os.remove(file_path)
    received_files.clear()

def main() -> None:
    updater = Updater(BOT_TOKEN)
    dispatcher = updater.dispatcher

    dispatcher.add_handler(CommandHandler('start', start))
    dispatcher.add_handler(MessageHandler(Filters.document | Filters.photo, handle_file))
    dispatcher.add_handler(CommandHandler('merge', merge_files))

    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
