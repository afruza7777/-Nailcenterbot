# ============================================================
# 💅 𝐀𝐬𝐲𝐚 𝐧𝐚𝐢𝐥𝐦𝐚𝐬𝐭𝐞𝐫
# TO'LIQ ISHLAYDIGAN TELEGRAM BOT
# Python 3.13 + python-telegram-bot 22.8
# ============================================================

!pip install -q -U python-telegram-bot==22.8 nest_asyncio

import nest_asyncio
nest_asyncio.apply()

import sqlite3
from datetime import datetime, timedelta
from zoneinfo import ZoneInfo

from telegram import Update, ReplyKeyboardMarkup, KeyboardButton
from telegram.ext import (
    Application,
    CommandHandler,
    MessageHandler,
    ContextTypes,
    filters
)

# ============================================================
# ⚙️ SOZLAMALAR
# ============================================================

TOKEN = "841447089:AAHlptCsaopbUyML-7okxR0aIMINVu1WArc"

ADMIN_ID = 8113310150

PHONE = "+998775249904"

ADDRESS = "📍 Xalq banki ro‘parasi, Sug‘diyona saloni"

LATITUDE = 39.7407546
LONGITUDE = 67.6378595

TZ = ZoneInfo("Asia/Tashkent")


# ============================================================
# 🗄 DATABASE
# ============================================================

conn = sqlite3.connect(
    "asya_nailmaster.db",
    check_same_thread=False
)

cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    user_id INTEGER PRIMARY KEY,
    points INTEGER DEFAULT 0
)
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS bookings (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER,
    name TEXT,
    phone TEXT,
    date TEXT,
    time TEXT,
    service TEXT,
    created_at TEXT
)
""")

conn.commit()


# ============================================================
# 👤 USER RO'YXATDAN O'TKAZISH
# ============================================================

def register_user(user_id):

    cursor.execute(
        """
        INSERT OR IGNORE INTO users
        (user_id, points)
        VALUES (?, 0)
        """,
        (user_id,)
    )

    conn.commit()


# ============================================================
# 🏠 ASOSIY MENU
# ============================================================

def main_menu():

    keyboard = [
        ["💅 Tirnoq xizmatlari"],
        ["📅 Bron qilish", "💰 Narxlar"],
        ["⭐ Mening ballarim"],
        ["📍 Manzil", "📞 Aloqa"]
    ]

    return ReplyKeyboardMarkup(
        keyboard,
        resize_keyboard=True
    )


# ============================================================
# 👑 ADMIN MENU
# ============================================================

def admin_menu():

    keyboard = [
        ["📋 Bugungi bronlar"],
        ["📋 Barcha bronlar"],
        ["⭐ Ball qo‘shish"],
        ["❌ Bronni bekor qilish"],
        ["🏠 Asosiy menyu"]
    ]

    return ReplyKeyboardMarkup(
        keyboard,
        resize_keyboard=True
    )


# ============================================================
# /START
# ============================================================

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):

    context.user_data.clear()

    user = update.effective_user

    register_user(user.id)

    await update.message.reply_text(
        f"""
Assalomu alaykum, {user.first_name}! 💅

✨ 𝐀𝐬𝐲𝐚 𝐧𝐚𝐢𝐥𝐦𝐚𝐬𝐭𝐞𝐫 ga xush kelibsiz! 💕

Kerakli bo‘limni tanlang 👇
""",
        reply_markup=main_menu()
    )


# ============================================================
# 💅 TIRNOQ XIZMATLARI
# ============================================================

async def nail_services(update, context):

    keyboard = [
        ["👩 Klientman"],
        ["📸 Modelman"],
        ["🧮 Narxni hisoblash"],
        ["⬅️ Orqaga"]
    ]

    await update.message.reply_text(
        "💅 Xizmat turini tanlang:",
        reply_markup=ReplyKeyboardMarkup(
            keyboard,
            resize_keyboard=True
        )
    )


# ============================================================
# 👩 KLIENT
# ============================================================

async def client_menu(update, context):

    await update.message.reply_text(
        """
👩 𝐊𝐋𝐈𝐄𝐍𝐓

💅 Manikyur — 160 000 so‘m
💎 Gel lak — 180 000 so‘m
💅 Narashka — 200 000 so‘m
🎨 Dizayn — +35 000 so‘m
""",
        reply_markup=main_menu()
    )


# ============================================================
# 📸 MODEL
# ============================================================

async def model_menu(update, context):

    await update.message.reply_text(
        """
📸 𝐌𝐎𝐃𝐄𝐋

💅 Manikyur — 80 000 so‘m
💎 Gel lak — 110 000 so‘m
💅 Narashka — 150 000 so‘m
🎨 Dizayn — +30 000 so‘m
""",
        reply_markup=main_menu()
    )


# ============================================================
# 💰 NARXLAR
# ============================================================

async def prices(update, context):

    keyboard = [
        ["🧮 Narxni hisoblash"],
        ["⬅️ Orqaga"]
    ]

    await update.message.reply_text(
        """
💰 𝐍𝐀𝐑𝐗𝐋𝐀𝐑

📸 MODEL

💅 Manikyur — 80 000 so‘m
💎 Gel lak — 110 000 so‘m
💅 Narashka — 150 000 so‘m
🎨 Dizayn — +30 000 so‘m


👩 KLIENT

💅 Manikyur — 160 000 so‘m
💎 Gel lak — 180 000 so‘m
💅 Narashka — 200 000 so‘m
🎨 Dizayn — +35 000 so‘m
""",
        reply_markup=ReplyKeyboardMarkup(
            keyboard,
            resize_keyboard=True
        )
    )


# ============================================================
# 🧮 KALKULYATOR BOSHLASH
# ============================================================

async def calculator_start(update, context):

    context.user_data["state"] = "calculator_type"

    keyboard = [
        ["👩 Klient narxi"],
        ["📸 Model narxi"],
        ["❌ Bekor qilish"]
    ]

    await update.message.reply_text(
        """
🧮 𝐍𝐀𝐑𝐗𝐍𝐈 𝐇𝐈𝐒𝐎𝐁𝐋𝐀𝐒𝐇

Avval narx turini tanlang:
""",
        reply_markup=ReplyKeyboardMarkup(
            keyboard,
            resize_keyboard=True
        )
    )


# ============================================================
# 🧮 KALKULYATOR TURI
# ============================================================

async def calculator_type(update, context):

    text = update.message.text

    if text == "👩 Klient narxi":

        context.user_data["calc_type"] = "client"

    elif text == "📸 Model narxi":

        context.user_data["calc_type"] = "model"

    else:

        return

    context.user_data["calc_total"] = 0
    context.user_data["calc_items"] = []
    context.user_data["state"] = "calculator"

    keyboard = [
        ["💅 Manikyur"],
        ["💎 Gel lak"],
        ["💅 Narashka"],
        ["🎨 Dizayn"],
        ["✅ Hisoblash"],
        ["🔄 Tozalash"],
        ["❌ Bekor qilish"]
    ]

    await update.message.reply_text(
        """
Kerakli xizmatlarni tanlang.

Bir nechta xizmat tanlashingiz mumkin 👇
""",
        reply_markup=ReplyKeyboardMarkup(
            keyboard,
            resize_keyboard=True
        )
    )


# ============================================================
# 🧮 KALKULYATOR
# ============================================================

async def calculator(update, context):

    text = update.message.text

    calc_type = context.user_data.get(
        "calc_type",
        "client"
    )

    if calc_type == "model":

        prices_dict = {
            "💅 Manikyur": 80000,
            "💎 Gel lak": 110000,
            "💅 Narashka": 150000,
            "🎨 Dizayn": 30000
        }

    else:

        prices_dict = {
            "💅 Manikyur": 160000,
            "💎 Gel lak": 180000,
            "💅 Narashka": 200000,
            "🎨 Dizayn": 35000
        }

    if text in prices_dict:

        price = prices_dict[text]

        context.user_data["calc_total"] += price

        context.user_data["calc_items"].append(
            text
        )

        total = context.user_data["calc_total"]

        await update.message.reply_text(
            f"""
✅ {text} qo‘shildi.

💰 Hozirgi jami:
{total:,} so‘m
"""
        )

    elif text == "🔄 Tozalash":

        context.user_data["calc_total"] = 0
        context.user_data["calc_items"] = []

        await update.message.reply_text(
            "🔄 Hisob tozalandi."
        )

    elif text == "✅ Hisoblash":

        total = context.user_data["calc_total"]

        if total == 0:

            await update.message.reply_text(
                "❌ Avval xizmat tanlang."
            )

            return

        items = "\n".join(
            "• " + item
            for item in context.user_data["calc_items"]
        )

        await update.message.reply_text(
            f"""
🧮 𝐇𝐈𝐒𝐎𝐁

{items}

━━━━━━━━━━━━

💰 Jami:
{total:,} so‘m
""",
            reply_markup=main_menu()
        )

        context.user_data.clear()

    elif text == "❌ Bekor qilish":

        context.user_data.clear()

        await update.message.reply_text(
            "❌ Hisoblash bekor qilindi.",
            reply_markup=main_menu()
        )


# ============================================================
# ⭐ MENING BALLARIM
# ============================================================

async def my_points(update, context):

    user_id = update.effective_user.id

    register_user(user_id)

    cursor.execute(
        """
        SELECT points
        FROM users
        WHERE user_id = ?
        """,
        (user_id,)
    )

    result = cursor.fetchone()

    points = result[0] if result else 0

    await update.message.reply_text(
        f"""
⭐ 𝐌𝐄𝐍𝐈𝐍𝐆 𝐁𝐀𝐋𝐋𝐀𝐑𝐈𝐌

Sizda:

⭐ {points} ball

Har tashrifdan keyin admin sizga ball qo‘shadi 💅
""",
        reply_markup=main_menu()
    )


# ============================================================
# 📍 MANZIL
# ============================================================

async def location(update, context):

    await update.message.reply_text(
        ADDRESS
    )

    await update.message.reply_location(
        latitude=LATITUDE,
        longitude=LONGITUDE
    )

    await update.message.reply_text(
        "📍 Sizni kutamiz! 💅",
        reply_markup=main_menu()
    )


# ============================================================
# 📞 ALOQA
# ============================================================

async def contact(update, context):

    await update.message.reply_text(
        f"""
📞 𝐀𝐋𝐎𝐐𝐀

Telefon:
{PHONE}
""",
        reply_markup=main_menu()
    )


# ============================================================
# 📅 BRON BOSHLASH
# ============================================================

async def booking_start(update, context):

    context.user_data.clear()

    context.user_data["state"] = "booking_time"

    keyboard = [
        ["12:00"],
        ["14:00"],
        ["16:00"],
        ["🕐 O‘zim istagan vaqtni yozaman"],
        ["❌ Bekor qilish"]
    ]

    await update.message.reply_text(
        """
📅 𝐁𝐑𝐎𝐍 𝐐𝐈𝐋𝐈𝐒𝐇

Vaqtni tanlang:
""",
        reply_markup=ReplyKeyboardMarkup(
            keyboard,
            resize_keyboard=True
        )
    )


# ============================================================
# 🕐 BRON VAQTI
# ============================================================

async def booking_time(update, context):

    text = update.message.text

    if text == "🕐 O‘zim istagan vaqtni yozaman":

        context.user_data["state"] = "custom_time"

        await update.message.reply_text(
            """
🕐 Istagan vaqtingizni yozing.

Masalan:
13:30
"""
        )

        return

    if text in ["12:00", "14:00", "16:00"]:

        context.user_data["booking_time"] = text

        context.user_data["state"] = "booking_date"

        await show_date_menu(
            update,
            context
        )


# ============================================================
# 📅 SANA MENU
# ============================================================

async def show_date_menu(update, context):

    keyboard = [
        ["📅 Bugun", "📅 Ertaga"],
        ["🗓 Boshqa sana"],
        ["❌ Bekor qilish"]
    ]

    await update.message.reply_text(
        "📅 Qaysi kunga bron qilamiz?",
        reply_markup=ReplyKeyboardMarkup(
            keyboard,
            resize_keyboard=True
        )
    )


# ============================================================
# 🕐 O'Z VAQTI
# ============================================================

async def custom_time(update, context):

    text = update.message.text.strip()

    try:

        datetime.strptime(
            text,
            "%H:%M"
        )

    except ValueError:

        await update.message.reply_text(
            """
❌ Vaqt noto‘g‘ri.

Masalan:
13:30
"""
        )

        return

    context.user_data["booking_time"] = text

    context.user_data["state"] = "booking_date"

    await show_date_menu(
        update,
        context
    )


# ============================================================
# 📅 SANA TANLASH
# ============================================================

async def booking_date(update, context):

    text = update.message.text

    now = datetime.now(TZ)

    if text == "📅 Bugun":

        date = now.strftime(
            "%d.%m.%Y"
        )

    elif text == "📅 Ertaga":

        date = (
            now + timedelta(days=1)
        ).strftime("%d.%m.%Y")

    elif text == "🗓 Boshqa sana":

        context.user_data["state"] = "custom_date"

        await update.message.reply_text(
            """
🗓 Sanani yozing.

Masalan:
25.10.2026
"""
        )

        return

    else:

        return

    context.user_data["booking_date"] = date

    await show_service_menu(
        update,
        context
    )


# ============================================================
# 🗓 O'Z SANASI
# ============================================================

async def custom_date(update, context):

    text = update.message.text.strip()

    try:

        selected = datetime.strptime(
            text,
            "%d.%m.%Y"
        )

        if selected.date() < datetime.now(TZ).date():

            await update.message.reply_text(
                "❌ O‘tgan sanaga bron qilib bo‘lmaydi."
            )

            return

    except ValueError:

        await update.message.reply_text(
            """
❌ Sana noto‘g‘ri.

Masalan:
25.10.2026
"""
        )

        return

    context.user_data["booking_date"] = text

    await show_service_menu(
        update,
        context
    )


# ============================================================
# 💅 BRON UCHUN XIZMAT
# ============================================================

async def show_service_menu(update, context):

    context.user_data["state"] = "booking_service"

    keyboard = [
        ["💅 Manikyur"],
        ["💎 Gel lak"],
        ["💅 Narashka"],
        ["🎨 Dizayn"],
        ["❌ Bekor qilish"]
    ]

    await update.message.reply_text(
        "💅 Qaysi xizmatni olmoqchisiz?",
        reply_markup=ReplyKeyboardMarkup(
            keyboard,
            resize_keyboard=True
        )
    )


# ============================================================
# 💅 XIZMAT TANLANDI
# ============================================================

async def booking_service(update, context):

    context.user_data["booking_service"] = (
        update.message.text
    )

    context.user_data["state"] = "booking_name"

    await update.message.reply_text(
        "👤 Ismingizni yozing:"
    )


# ============================================================
# 👤 ISM
# ============================================================

async def booking_name(update, context):

    name = update.message.text.strip()

    if len(name) < 2:

        await update.message.reply_text(
            "❌ Ismni to‘g‘ri yozing."
        )

        return

    context.user_data["booking_name"] = name

    context.user_data["state"] = "booking_phone"

    keyboard = [
        [
            KeyboardButton(
                "📞 Telefon raqamimni yuborish",
                request_contact=True
            )
        ],
        ["❌ Bekor qilish"]
    ]

    await update.message.reply_text(
        "📞 Telefon raqamingizni yuboring:",
        reply_markup=ReplyKeyboardMarkup(
            keyboard,
            resize_keyboard=True,
            one_time_keyboard=True
        )
    )


# ============================================================
# 💾 BRONNI SAQLASH
# ============================================================

async def save_booking(
    update,
    context,
    phone
):

    user_id = update.effective_user.id

    name = context.user_data.get(
        "booking_name",
        ""
    )

    date = context.user_data.get(
        "booking_date",
        ""
    )

    time = context.user_data.get(
        "booking_time",
        ""
    )

    service = context.user_data.get(
        "booking_service",
        ""
    )

    # Vaqt bandligini tekshirish

    cursor.execute(
        """
        SELECT id
        FROM bookings
        WHERE date = ?
        AND time = ?
        """,
        (date, time)
    )

    if cursor.fetchone():

        await update.message.reply_text(
            f"""
❌ Afsus!

📅 {date}
🕐 {time}

Bu vaqt allaqachon band.

Iltimos, boshqa vaqt tanlang.
""",
            reply_markup=main_menu()
        )

        context.user_data.clear()

        return

    created_at = datetime.now(
        TZ
    ).strftime(
        "%d.%m.%Y %H:%M"
    )

    cursor.execute(
        """
        INSERT INTO bookings
        (
            user_id,
            name,
            phone,
            date,
            time,
            service,
            created_at
        )
        VALUES (?, ?, ?, ?, ?, ?, ?)
        """,
        (
            user_id,
            name,
            phone,
            date,
            time,
            service,
            created_at
        )
    )

    conn.commit()

    booking_id = cursor.lastrowid

    # Admin xabari

    admin_text = f"""
🔔 𝐘𝐀𝐍𝐆𝐈 𝐁𝐑𝐎𝐍!

📌 Bron №: {booking_id}

👤 Ism: {name}
📞 Telefon: {phone}
📅 Sana: {date}
🕐 Vaqt: {time}
💅 Xizmat: {service}

🆔 Telegram ID:
{user_id}
"""

    try:

        await context.bot.send_message(
            chat_id=ADMIN_ID,
            text=admin_text
        )

    except Exception as e:

        print(
            "Admin xabar yuborilmadi:",
            e
        )

    # Klientga xabar

    await update.message.reply_text(
        f"""
✅ 𝐁𝐑𝐎𝐍 𝐌𝐔𝐕𝐀𝐅𝐅𝐀𝐐𝐈𝐘𝐀𝐓𝐋𝐈 𝐐𝐀𝐁𝐔𝐋 𝐐𝐈𝐋𝐈𝐍𝐃𝐈!

📌 Bron №: {booking_id}

👤 Ism: {name}
📞 Telefon: {phone}
📅 Sana: {date}
🕐 Vaqt: {time}
💅 Xizmat: {service}

Tez orada siz bilan bog‘lanamiz 💅
""",
        reply_markup=main_menu()
    )

    context.user_data.clear()


# ============================================================
# 📞 TELEFON CONTACT
# ============================================================

async def contact_handler(update, context):

    if context.user_data.get(
        "state"
    ) != "booking_phone":

        return

    phone = update.message.contact.phone_number

    await save_booking(
        update,
        context,
        phone
    )


# ============================================================
# 📞 TELEFONNI YOZIB YUBORSA
# ============================================================

async def typed_phone(update, context):

    phone = update.message.text.strip()

    if len(phone) < 7:

        await update.message.reply_text(
            "❌ Telefon raqamini to‘g‘ri yozing."
        )

        return

    await save_booking(
        update,
        context,
        phone
    )


# ============================================================
# 👑 ADMIN TEKSHIRISH
# ============================================================

def is_admin(update):

    return (
        update.effective_user.id
        == ADMIN_ID
    )


# ============================================================
# 👑 /ADMIN
# ============================================================

async def admin_command(update, context):

    if not is_admin(update):

        await update.message.reply_text(
            "⛔ Siz admin emassiz."
        )

        return

    context.user_data.clear()

    await update.message.reply_text(
        """
👑 𝐀𝐃𝐌𝐈𝐍 𝐏𝐀𝐍𝐄𝐋

Kerakli bo‘limni tanlang:
""",
        reply_markup=admin_menu()
    )


# ============================================================
# 📋 BUGUNGI BRONLAR
# ============================================================

async def today_bookings(update, context):

    if not is_admin(update):

        return

    today = datetime.now(
        TZ
    ).strftime(
        "%d.%m.%Y"
    )

    cursor.execute(
        """
        SELECT
            id,
            name,
            phone,
            time,
            service,
            user_id
        FROM bookings
        WHERE date = ?
        ORDER BY time
        """,
        (today,)
    )

    rows = cursor.fetchall()

    if not rows:

        await update.message.reply_text(
            f"""
📋 𝐁𝐔𝐆𝐔𝐍𝐆𝐈 𝐁𝐑𝐎𝐍𝐋𝐀𝐑

📅 {today}

Hozircha bron yo‘q.
"""
        )

        return

    text = (
        f"📋 𝐁𝐔𝐆𝐔𝐍𝐆𝐈 𝐁𝐑𝐎𝐍𝐋𝐀𝐑\n"
        f"📅 {today}\n\n"
    )

    for row in rows:

        (
            booking_id,
            name,
            phone,
            time,
            service,
            user_id
        ) = row

        text += (
            f"№{booking_id}\n"
            f"👤 {name}\n"
            f"📞 {phone}\n"
            f"🕐 {time}\n"
            f"💅 {service}\n"
            f"🆔 {user_id}\n"
            f"────────────\n"
        )

    await update.message.reply_text(
        text
    )


# ============================================================
# 📋 BARCHA BRONLAR
# ============================================================

async def all_bookings(update, context):

    if not is_admin(update):

        return

    cursor.execute(
        """
        SELECT
            id,
            name,
            phone,
            date,
            time,
            service,
            user_id
        FROM bookings
        ORDER BY id DESC
        LIMIT 100
        """
    )

    rows = cursor.fetchall()

    if not rows:

        await update.message.reply_text(
            "📋 Hozircha bronlar yo‘q."
        )

        return

    text = "📋 𝐁𝐀𝐑𝐂𝐇𝐀 𝐁𝐑𝐎𝐍𝐋𝐀𝐑\n\n"

    for row in rows:

        (
            booking_id,
            name,
            phone,
            date,
            time,
            service,
            user_id
        ) = row

        text += (
            f"№{booking_id}\n"
            f"👤 {name}\n"
            f"📞 {phone}\n"
            f"📅 {date}\n"
            f"🕐 {time}\n"
            f"💅 {service}\n"
            f"🆔 {user_id}\n"
            f"────────────\n"
        )

    await update.message.reply_text(
        text
    )


# ============================================================
# ⭐ BALL QO'SHISH
# ============================================================

async def points_start(update, context):

    if not is_admin(update):

        return

    context.user_data[
        "admin_state"
    ] = "points_user"

    await update.message.reply_text(
        """
⭐ 𝐁𝐀𝐋𝐋 𝐐𝐎‘𝐒𝐇𝐈𝐒𝐇

Klientning Telegram ID sini yozing:
"""
    )


async def points_user(update, context):

    try:

        target_id = int(
            update.message.text.strip()
        )

    except ValueError:

        await update.message.reply_text(
            "❌ ID faqat raqam bo‘lishi kerak."
        )

        return

    context.user_data[
        "points_target"
    ] = target_id

    context.user_data[
        "admin_state"
    ] = "points_amount"

    await update.message.reply_text(
        """
⭐ Nechta ball qo‘shamiz?

Masalan:
1
"""
    )


async def points_amount(update, context):

    try:

        amount = int(
            update.message.text.strip()
        )

        if amount <= 0:

            raise ValueError

    except ValueError:

        await update.message.reply_text(
            "❌ Ball sonini to‘g‘ri yozing."
        )

        return

    target_id = context.user_data[
        "points_target"
    ]

    register_user(target_id)

    cursor.execute(
        """
        UPDATE users
        SET points = points + ?
        WHERE user_id = ?
        """,
        (
            amount,
            target_id
        )
    )

    conn.commit()

    cursor.execute(
        """
        SELECT points
        FROM users
        WHERE user_id = ?
        """,
        (target_id,)
    )

    total = cursor.fetchone()[0]

    await update.message.reply_text(
        f"""
✅ 𝐁𝐀𝐋𝐋 𝐐𝐎‘𝐒𝐇𝐈𝐋𝐃𝐈!

🆔 ID: {target_id}

⭐ Qo‘shildi: {amount}
⭐ Jami: {total}
"""
    )

    try:

        await context.bot.send_message(
            chat_id=target_id,
            text=f"""
🎉 Sizga ⭐ {amount} ball qo‘shildi!

⭐ Jami ballingiz:
{total}
"""
        )

    except Exception:

        pass

    context.user_data.clear()


# ============================================================
# ❌ BRONNI BEKOR QILISH
# ============================================================

async def cancel_booking_start(update, context):

    if not is_admin(update):

        return

    context.user_data[
        "admin_state"
    ] = "cancel_booking"

    await update.message.reply_text(
        """
❌ 𝐁𝐑𝐎𝐍𝐍𝐈 𝐁𝐄𝐊𝐎𝐑 𝐐𝐈𝐋𝐈𝐒𝐇

Bron raqamini yozing.

Masalan:
15
"""
    )


async def cancel_booking(update, context):

    try:

        booking_id = int(
            update.message.text.strip()
        )

    except ValueError:

        await update.message.reply_text(
            "❌ Bron raqamini to‘g‘ri yozing."
        )

        return

    cursor.execute(
        """
        SELECT
            name,
            user_id,
            date,
            time
        FROM bookings
        WHERE id = ?
        """,
        (booking_id,)
    )

    booking = cursor.fetchone()

    if not booking:

        await update.message.reply_text(
            "❌ Bunday bron topilmadi."
        )

        return

    (
        name,
        user_id,
        date,
        time
    ) = booking

    cursor.execute(
        """
        DELETE FROM bookings
        WHERE id = ?
        """,
        (booking_id,)
    )

    conn.commit()

    await update.message.reply_text(
        f"""
✅ 𝐁𝐑𝐎𝐍 𝐁𝐄𝐊𝐎𝐑 𝐐𝐈𝐋𝐈𝐍𝐃𝐈

№{booking_id}

👤 {name}
📅 {date}
🕐 {time}
"""
    )

    try:

        await context.bot.send_message(
            chat_id=user_id,
            text=f"""
❌ Sizning №{booking_id} broningiz bekor qilindi.
"""
        )

    except Exception:

        pass

    context.user_data.clear()


# ============================================================
# 💬 ASOSIY MESSAGE HANDLER
# ============================================================

async def message_handler(update, context):

    text = update.message.text

    # --------------------------------------------------------
    # ADMIN HOLATLARI
    # --------------------------------------------------------

    if is_admin(update):

        admin_state = context.user_data.get(
            "admin_state"
        )

        if admin_state == "points_user":

            await points_user(
                update,
                context
            )

            return

        if admin_state == "points_amount":

            await points_amount(
                update,
                context
            )

            return

        if admin_state == "cancel_booking":

            await cancel_booking(
                update,
                context
            )

            return

    # --------------------------------------------------------
    # USER HOLATLARI
    # --------------------------------------------------------

    state = context.user_data.get(
        "state"
    )

    if state == "booking_time":

        await booking_time(
            update,
            context
        )

        return

    if state == "custom_time":

        await custom_time(
            update,
            context
        )

        return

    if state == "booking_date":

        await booking_date(
            update,
            context
        )

        return

    if state == "custom_date":

        await custom_date(
            update,
            context
        )

        return

    if state == "booking_service":

        await booking_service(
            update,
            context
        )

        return

    if state == "booking_name":

        await booking_name(
            update,
            context
        )

        return

    if state == "booking_phone":

        await typed_phone(
            update,
            context
        )

        return

    if state == "calculator_type":

        await calculator_type(
            update,
            context
        )

        return

    if state == "calculator":

        await calculator(
            update,
            context
        )

        return

    # --------------------------------------------------------
    # ASOSIY TUGMALAR
    # --------------------------------------------------------

    if text == "💅 Tirnoq xizmatlari":

        await nail_services(
            update,
            context
        )

    elif text == "👩 Klientman":

        await client_menu(
            update,
            context
        )

    elif text == "📸 Modelman":

        await model_menu(
            update,
            context
        )

    elif text == "📅 Bron qilish":

        await booking_start(
            update,
            context
        )

    elif text == "💰 Narxlar":

        await prices(
            update,
            context
        )

    elif text == "⭐ Mening ballarim":

        await my_points(
            update,
            context
        )

    elif text == "📍 Manzil":

        await location(
            update,
            context
        )

    elif text == "📞 Aloqa":

        await contact(
            update,
            context
        )

    elif text == "🧮 Narxni hisoblash":

        await calculator_start(
            update,
            context
        )

    # --------------------------------------------------------
    # ADMIN TUGMALARI
    # --------------------------------------------------------

    elif text == "📋 Bugungi bronlar":

        await today_bookings(
            update,
            context
        )

    elif text == "📋 Barcha bronlar":

        await all_bookings(
            update,
            context
        )

    elif text == "⭐ Ball qo‘shish":

        await points_start(
            update,
            context
        )

    elif text == "❌ Bronni bekor qilish":

        await cancel_booking_start(
            update,
            context
        )

    # --------------------------------------------------------
    # ORQAGA / BEKOR
    # --------------------------------------------------------

    elif text in [
        "⬅️ Orqaga",
        "❌ Bekor qilish"
    ]:

        context.user_data.clear()

        await update.message.reply_text(
            "🏠 Asosiy menyu",
            reply_markup=main_menu()
        )

    elif text == "🏠 Asosiy menyu":

        context.user_data.clear()

        await update.message.reply_text(
            "🏠 Asosiy menyu",
            reply_markup=main_menu()
        )


# ============================================================
# ❗ ERROR
# ============================================================

async def error_handler(update, context):

    print(
        "BOT XATOSI:",
        repr(context.error)
    )


# ============================================================
# 🚀 BOTNI ISHGA TUSHIRISH
# ============================================================

app = (
    Application
    .builder()
    .token(TOKEN)
    .build()
)

app.add_handler(
    CommandHandler(
        "start",
        start
    )
)

app.add_handler(
    CommandHandler(
        "admin",
        admin_command
    )
)

app.add_handler(
    MessageHandler(
        filters.CONTACT,
        contact_handler
    )
)

app.add_handler(
    MessageHandler(
        filters.TEXT & ~filters.COMMAND,
        message_handler
    )
)

app.add_error_handler(
    error_handler
)

print("======================================")
print("💅 𝐀𝐬𝐲𝐚 𝐧𝐚𝐢𝐥𝐦𝐚𝐬𝐭𝐞𝐫")
print("🤖 BOT ISHGA TUSHMOQDA...")
print("======================================")

app.run_polling(
    drop_pending_updates=True
)
