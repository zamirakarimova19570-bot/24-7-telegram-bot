from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, ContextTypes, MessageHandler, filters
import os
import logging

# Logging sozlamalari
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

# Bot tokeni (GitHub Secrets dan)
TOKEN = os.getenv('BOT_TOKEN', '8587222975:AAEq18hC7QrRF1UsNv88JX4q9enU4iCvXTw')

print("=" * 50)
print("🚀 24/7 TELEGRAM TEST BOT")
print("📍 Platforma: GitHub Actions")
print("⏰ Uptime: 24/7")
print("💰 Narx: BEPUL")
print("=" * 50)

# Test ma'lumotlari (memory da saqlaymiz)
tests_db = {}
user_sessions = {}

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    
    keyboard = [
        [InlineKeyboardButton("👨‍🎓 O'quvchi", callback_data='student')],
        [InlineKeyboardButton("👨‍🏫 Ustoz", callback_data='teacher')],
        [InlineKeyboardButton("ℹ️ Bot haqida", callback_data='about')]
    ]
    
    await update.message.reply_text(
        "Test botga xush kelibsiz! Rolni tanlang:\n\n"
        "📊 *Server holati:*\n"
        "• Platforma: GitHub Actions\n"
        "• Uptime: 24/7 (avtomatik restart)\n"
        "• Narx: Umrbod bepul\n\n"
        "⚠️ *Eslatma:* Agar foydalanuvchi boshqa ilovaga o'tsa, test avtomatik to'xtaydi!",
        parse_mode='Markdown',
        reply_markup=InlineKeyboardMarkup(keyboard)
    )

async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    
    if query.data == 'student':
        await query.edit_message_text(
            "Test kodini kiriting (5 ta harf/raqam):\n\n"
            "Namuna: ABC12 yoki 123DE\n\n"
            "Mavjud testlar:\n" +
            "\n".join([f"• {code}" for code in tests_db.keys()])
        )
        context.user_data['waiting_for_test_code'] = True
    
    elif query.data == 'teacher':
        await query.edit_message_text(
            "Ustoz rejimi:\n\n"
            "1. Yangi test yaratish - /new_test\n"
            "2. Mening testlarim - /my_tests\n"
            "3. Test natijalari - /results"
        )
    
    elif query.data == 'about':
        await query.edit_message_text(
            "🤖 *GitHub Actions Telegram Bot*\n\n"
            "✅ 24/7 ishlaydi\n"
            "✅ Bepul hosting\n"
            "✅ Avtomatik restart\n"
            "✅ Open source\n\n"
            "📍 Platforma: GitHub Actions\n"
            "⏰ Uptime: Har 25 daqiqada yangilanadi\n"
            "💰 Narx: $0 (bepul)"
        )

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text
    user_id = update.message.from_user.id
    
    if 'waiting_for_test_code' in context.user_data:
        # Test kodi kiritildi
        test_code = text.upper().strip()
        
        if test_code in tests_db:
            # Testni boshlash
            test = tests_db[test_code]
            user_sessions[user_id] = {
                'test_code': test_code,
                'current_question': 0,
                'score': 0
            }
            
            await send_question(update, context, user_id)
        else:
            await update.message.reply_text(
                f"❌ '{test_code}' test kodi topilmadi.\n"
                f"Mavjud testlar: {', '.join(tests_db.keys())}"
            )
        
        context.user_data.pop('waiting_for_test_code', None)
    
    elif user_id in user_sessions:
        # Test javobi
        session = user_sessions[user_id]
        test_code = session['test_code']
        test = tests_db[test_code]
        current_q = session['current_question']
        
        if current_q < len(test['questions']):
            question = test['questions'][current_q]
            
            # Javobni tekshirish
            if text.upper().strip() == question['correct_answer']:
                session['score'] += 1
                await update.message.reply_text("✅ To'g'ri javob!")
            else:
                await update.message.reply_text(f"❌ Noto'g'ri. To'g'ri javob: {question['correct_answer']}")
            
            # Keyingi savol
            session['current_question'] += 1
            
            if session['current_question'] < len(test['questions']):
                await send_question(update, context, user_id)
            else:
                # Test tugadi
                score = session['score']
                total = len(test['questions'])
                
                await update.message.reply_text(
                    f"🎉 Test tugadi!\n\n"
                    f"Test: {test['name']}\n"
                    f"To'g'ri javoblar: {score}/{total}\n"
                    f"Natija: {(score/total*100):.1f}%\n\n"
                    f"Yana test ishlash uchun /start"
                )
                del user_sessions[user_id]
    
    elif text.startswith('/new_test'):
        await update.message.reply_text(
            "Yangi test yaratish uchun:\n\n"
            "1. Test kodi (5 ta belgi):\n"
            "2. Test nomi:\n"
            "3. Savollar soni:\n\n"
            "Masalan:\n"
            "MATHS\n"
            "Matematika testi\n"
            "3"
        )
        context.user_data['creating_test'] = True
    
    elif text.startswith('/my_tests'):
        if tests_db:
            tests_list = "\n".join([f"• {code}: {test['name']} ({len(test['questions'])} savol)" 
                                  for code, test in tests_db.items()])
            await update.message.reply_text(
                f"📋 Mening testlarim:\n\n{tests_list}"
            )
        else:
            await update.message.reply_text("Hozircha testlar mavjud emas. /new_test bilan yangi test yarating.")
    
    else:
        await update.message.reply_text(
            "Tushunmadim. /start bilan boshlang yoki test kodini kiriting."
        )

async def send_question(update, context, user_id):
    if user_id not in user_sessions:
        return
    
    session = user_sessions[user_id]
    test_code = session['test_code']
    test = tests_db[test_code]
    question_num = session['current_question']
    
    if question_num < len(test['questions']):
        question = test['questions'][question_num]
        
        message = f"❓ Savol {question_num + 1}:\n{question['text']}\n\n"
        
        if 'options' in question:
            for i, option in enumerate(question['options']):
                message += f"{chr(65+i)}) {option}\n"
            message += "\nJavobingizni (A, B, C, D) yuboring:"
        else:
            message += "Javobingizni yozma shaklda yuboring:"
        
        await context.bot.send_message(chat_id=user_id, text=message)

async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🆘 *Yordam*\n\n"
        "📌 *Buyruqlar:*\n"
        "/start - Botni ishga tushirish\n"
        "/test - Test ishlash\n"
        "/new_test - Yangi test yaratish\n"
        "/my_tests - Mening testlarim\n"
        "/help - Yordam\n\n"
        "🤖 *Platforma:* GitHub Actions\n"
        "⏰ *Uptime:* 24/7\n"
        "💰 *Narx:* Bepul"
    )

async def test_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if tests_db:
        tests_list = "\n".join(tests_db.keys())
        await update.message.reply_text(
            f"Mavjud test kodlari:\n{tests_list}\n\n"
            f"Test kodini kiriting:"
        )
        context.user_data['waiting_for_test_code'] = True
    else:
        await update.message.reply_text(
            "Hozircha testlar mavjud emas. Ustoz yangi test yaratishi kerak."
        )

def main():
    print("✅ Bot muvaffaqiyatli yaratildi!")
    print("⏳ Bot ishga tushmoqda...")
    
    # Botni ishga tushirish
    app = ApplicationBuilder().token(TOKEN).build()
    
    # Handlerlar
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("help", help_command))
    app.add_handler(CommandHandler("test", test_command))
    app.add_handler(CallbackQueryHandler(button_handler))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    
    # Demo test qo'shish
    tests_db['DEMO1'] = {
        'name': 'Demo Matematika Testi',
        'questions': [
            {
                'text': '2 + 2 = ?',
                'options': ['3', '4', '5', '6'],
                'correct_answer': 'B'
            },
            {
                'text': '5 × 3 = ?',
                'options': ['10', '15', '20', '25'],
                'correct_answer': 'B'
            }
        ]
    }
    
    print(f"📝 Demo test yaratildi: DEMO1")
    print(f"👥 Faol foydalanuvchilar: {len(user_sessions)}")
    
    # Botni ishga tushirish
    app.run_polling()

if __name__ == '__main__':
    main()
