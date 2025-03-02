import random
import time
from telegram import Update
from telegram.ext import Updater, CommandHandler, CallbackContext

# ✅ التوكن الخاص بالبوت
TOKEN = '7562916060:AAFGk2KLQdUK_KQ3kZspSOyLkO-XN1u-5OI'

# ✅ معرف المسؤول (أنت فقط)
ADMIN_ID = 6483902212

# ✅ قائمة حسابات Crunchyroll
accounts = [
    {"email": "viniciuspassos132@outlook.com", "password": "frota8049", "users": 0},
    {"email": "josephflores1109@gmail.com", "password": "steven1109", "users": 0},
    {"email": "andrewmatts1998@gmail.com", "password": "Armatts031398", "users": 0},
    {"email": "jhoansebastianmendezayala@gmail.com", "password": "jhonanderson1092", "users": 0},
    {"email": "pedroespinal0224@gmail.com", "password": "pedro021423", "users": 0},
]

# ✅ سجل الطلبات للمستخدمين
user_last_request = {}
user_accounts = {}

# ✅ فترة الانتظار (48 ساعة)
WAIT_TIME = 172800

# ✅ رسائل مضحكة
funny_responses = [
    "أهوه الحساب، خذه قبل ما أغيّر رأيي! 😏",
    "هذا الحساب طازة من الفرن! 🍞",
    "لا تقول إني ما دلعتك! 🎁",
    "خذ الحساب بسرعة قبل ما ينحظر! 🚀",
    "يلا، روح شوف الأنمي وتذكرني! 🍿",
    "انتبه، هذا الحساب فيه سحر الأنمي! ✨",
]

# ✅ رسالة عند نفاد الحسابات
no_accounts_message = "معليش يا نجم، الحسابات خلصت مؤقتًا! 😢 ارجع لاحقًا."

# ✅ بدء البوت
def start(update: Update, context: CallbackContext) -> None:
    update.message.reply_text(
        "👋 مرحبًا! هذا بوت توزيع حسابات **Crunchyroll**!\n"
        "🟠 اطلب حساب باستخدام الأمر: `/crunchyroll`\n"
        "📋 تحقق من حساباتك: `/myaccounts`\n"
        "ℹ️ لمعرفة المزيد: `/info`\n"
        "📌 ملاحظة: حساب واحد كل **48 ساعة**!"
    )

# ✅ إرسال حساب Crunchyroll
def send_crunchyroll(update: Update, context: CallbackContext) -> None:
    user_id = update.message.from_user.id
    user_name = update.message.from_user.first_name

    # ✅ المسؤول يحصل على وصول غير محدود
    if user_id == ADMIN_ID:
        available_accounts = [acc for acc in accounts]
    else:
        # تحقق من الوقت إذا كان المستخدم عادي
        now = time.time()
        if user_id in user_last_request and now - user_last_request[user_id] < WAIT_TIME:
            update.message.reply_text(f"⏳ يا {user_name}، اصبر... لازم تمر 48 ساعة قبل ما تاخذ حساب ثاني!")
            return

        # اختيار حساب متاح (أقل من 4 مستخدمين)
        available_accounts = [acc for acc in accounts if acc["users"] < 4]

    # ✅ إذا لم يكن هناك حسابات متاحة
    if not available_accounts:
        update.message.reply_text(no_accounts_message)
        return

    # ✅ إرسال حساب عشوائي وتحديث السجل
    account = random.choice(available_accounts)

    # ✅ لا يتم احتساب المسؤول ضمن عدد المستخدمين
    if user_id != ADMIN_ID:
        account["users"] += 1
        user_last_request[user_id] = time.time()

        # إذا استُخدم الحساب 4 مرات، نحذفه
        if account["users"] >= 4:
            accounts.remove(account)

    # ✅ تخزين الحساب في سجل المستخدم
    if user_id not in user_accounts:
        user_accounts[user_id] = []
    user_accounts[user_id].append(account)

    funny_message = random.choice(funny_responses)

    update.message.reply_text(
        f"🎉 {funny_message}\n\n📧 **الإيميل:** `{account['email']}`\n🔑 **الباسورد:** `{account['password']}`",
        parse_mode="Markdown"
    )

# ✅ عرض الحسابات الخاصة بالمستخدم
def my_accounts(update: Update, context: CallbackContext) -> None:
    user_id = update.message.from_user.id

    if user_id not in user_accounts or not user_accounts[user_id]:
        update.message.reply_text("📭 ما عندك أي حسابات حتى الآن. اطلب حساب بـ `/crunchyroll`!")
        return

    message = "📋 **حساباتك السابقة:**\n\n"
    for i, account in enumerate(user_accounts[user_id], 1):
        message += f"🔢 {i} - 📧 `{account['email']}` | 🔑 `{account['password']}`\n"

    update.message.reply_text(message, parse_mode="Markdown")

# ✅ معلومات حول البوت
def info(update: Update, context: CallbackContext) -> None:
    update.message.reply_text(
        "ℹ️ **معلومات حول البوت:**\n"
        "🟠 **/crunchyroll** - لطلب حساب جديد (حساب واحد كل 48 ساعة).\n"
        "📋 **/myaccounts** - لعرض الحسابات التي حصلت عليها.\n"
        "📌 الحسابات محدودة، أسرع واحصل على واحد قبل ما ينتهي المخزون!"
    )

# ✅ أوامر المسؤول (مخفية عن غير المسؤول)
def admin(update: Update, context: CallbackContext) -> None:
    user_id = update.message.from_user.id

    if user_id != ADMIN_ID:
        return

    remaining_accounts = len([acc for acc in accounts if acc["users"] < 4])

    update.message.reply_text(
        f"📊 **وضع البوت:**\n"
        f"🔢 الحسابات المتاحة: {remaining_accounts}\n"
        f"/reset - إعادة ضبط جميع الاستخدامات\n"
        f"/add - إضافة حساب جديد (صيغة: /add email password)"
    )

# ✅ إعادة ضبط الاستخدامات
def reset(update: Update, context: CallbackContext) -> None:
    user_id = update.message.from_user.id

    if user_id != ADMIN_ID:
        return

    for account in accounts:
        account["users"] = 0
    user_last_request.clear()
    user_accounts.clear()

    update.message.reply_text("✅ تم إعادة ضبط جميع الاستخدامات!")

# ✅ إضافة حساب جديد
def add_account(update: Update, context: CallbackContext) -> None:
    user_id = update.message.from_user.id

    if user_id != ADMIN_ID:
        return

    try:
        _, email, password = update.message.text.split(maxsplit=2)
        accounts.append({"email": email, "password": password, "users": 0})
        update.message.reply_text(f"✅ تم إضافة الحساب بنجاح:\n📧 {email}")
    except ValueError:
        update.message.reply_text("❌ الصيغة الصحيحة: `/add email password`")

# ✅ بدء تشغيل البوت
def main():
    updater = Updater(TOKEN)
    dp = updater.dispatcher

    # أوامر البوت
    dp.add_handler(CommandHandler("start", start))
    dp.add_handler(CommandHandler("crunchyroll", send_crunchyroll))
    dp.add_handler(CommandHandler("myaccounts", my_accounts))
    dp.add_handler(CommandHandler("info", info))

    # أوامر الإدارة (مخفية)
    dp.add_handler(CommandHandler("admin", admin))
    dp.add_handler(CommandHandler("reset", reset))
    dp.add_handler(CommandHandler("add", add_account))

    print("✅ البوت شغّال... استمتع بتوزيع الحسابات!")
    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()