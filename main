import re
from aiogram import F, Router, Bot, Dispatcher
from aiogram.types import Message, InlineKeyboardButton
from aiogram.utils.keyboard import InlineKeyboardBuilder
from aiogram.filters import CommandStart

# Токен бота
BOT_TOKEN = "8349157545:AAH5oGQE0dI-GRYEO-m9gHulzsWRYgF6Gms"
# ID администратора
ADMIN_ID = 5787083558

router = Router()

# Храним состояние пользователей
waiting_for_video = set()

# Словарь для хранения соответствия username -> user_id (chat_id)
username_to_id = {}

@router.message(CommandStart())
async def cmd_start(message: Message):
    # Сохраняем username и user_id, если у пользователя есть username
    if message.from_user.username:
        username_to_id[message.from_user.username] = message.from_user.id

    if message.from_user.id != ADMIN_ID:
        kb = InlineKeyboardBuilder()
        kb.button(text="Участвовать", callback_data="participate")
        await message.answer(
            text="Привет! Если хочешь участвовать в конкурсе статистики, то нажимай на кнопку ниже.",
            reply_markup=kb.as_markup()
        )
    else:
        await message.answer("Привет, администратор!")

@router.callback_query(F.data == "participate")
async def on_participate(callback_query):
    await callback_query.answer()
    await callback_query.message.answer(
        text="Отправь видео, которое должно содержать: момент, когда вы заходите на сервер, открывайте статистику и показывайте необходимые данные, для участия в определённой номинации (Можно участвовать сразу во всех!) ОБЯЗАТЕЛЬНО должен быть виден ваш ник на видео, чтобы мы могли определить вас))"
    )
    waiting_for_video.add(callback_query.from_user.id)

@router.message(F.video, F.from_user.id.in_(waiting_for_video))
async def handle_video(message: Message):
    # Сохраняем username, если он есть, перед отправкой видео админу
    if message.from_user.username:
        username_to_id[message.from_user.username] = message.from_user.id

    await bot.forward_message(chat_id=ADMIN_ID, from_chat_id=message.chat.id, message_id=message.message_id)
    await message.answer("Видео успешно отправлено администратору!")
    waiting_for_video.discard(message.from_user.id)

@router.message(F.from_user.id.in_(waiting_for_video))
async def handle_non_video(message: Message):
    await message.answer("Пожалуйста, отправьте видео.")

# === Обработка сообщений администратора ===
@router.message(F.from_user.id == ADMIN_ID, F.text)
async def handle_admin_message(message: Message):
    text = message.text.strip()
    if text.upper().startswith("ОТВЕТ:"):
        match = re.search(r"ОТВЕТ:\s*(.*?),\s*@(\w+)", text, re.IGNORECASE | re.DOTALL)
        if match:
            reply_text = match.group(1).strip()
            username = match.group(2)

            user_id = username_to_id.get(username)
            if user_id:
                try:
                    await bot.send_message(chat_id=user_id, text=reply_text)
                    await message.answer(f"Сообщение пользователю @{username} отправлено.")
                except Exception as e:
                    await message.answer(f"Не удалось отправить сообщение пользователю @{username}: {e}")
            else:
                await message.answer(f"Пользователь @{username} не найден. Возможно, он не взаимодействовал с ботом.")
        else:
            await message.answer("Неправильный формат сообщения. Используйте: ОТВЕТ: текст, @username")
    # Игнорируем другие сообщения администратора

if __name__ == "__main__":
    bot = Bot(token=BOT_TOKEN)
    dp = Dispatcher()
    dp.include_router(router)
    dp.run_polling(bot)
