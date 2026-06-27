import os
import telebot
from telebot import types

# Токен берётся из переменной окружения Render (безопасно)
TOKEN = os.getenv("TOKEN")
if not TOKEN:
    # Если запускаешь локально, вставь токен сюда и закомментируй строку выше
    TOKEN = "ТВОЙ_ТОКЕН_ОТ_BOTFATHER"

CHANNEL = "@yandex_market_gid"
CHANNEL_ID = -1004209318688  # Твой ID канала

bot = telebot.TeleBot(TOKEN)

def is_subscribed(user_id: int) -> bool:
    """Возвращает True, если пользователь подписан на канал."""
    try:
        member = bot.get_chat_member(CHANNEL_ID, user_id)
        return member.status not in ["left", "kicked"]
    except Exception as e:
        print(f"Ошибка проверки подписки для {user_id}: {e}")
        return False

@bot.message_handler(commands=['start'])
def start_command(message: telebot.types.Message):
    user_id = message.from_user.id

    if is_subscribed(user_id):
        text = (
            "Ура, ты с нами! ♥\n\n"
            "Ты участвуешь в розыгрыше сертификатов Яндекс Маркета:\n"
            "🥇 10 000 ₽\n"
            "🥈 5 000 ₽\n"
            "🥉 2 000 ₽\n\n"
            "Победителей определю 15 числа случайным образом и объявлю в канале.\n\n"
            "Чтобы увеличить шансы, ставь реакции на посты и пиши комментарии.\n"
            "Спасибо, что ты со мной! 🌸"
        )
        bot.send_message(message.chat.id, text)
    else:
        markup = types.InlineKeyboardMarkup(row_width=1)
        btn_subscribe = types.InlineKeyboardButton(
            "Подписаться на канал ♥",
            url="https://t.me/yandex_market_gid"
        )
        btn_check = types.InlineKeyboardButton(
            "Проверить подписку",
            callback_data="check_subscription"
        )
        markup.add(btn_subscribe, btn_check)

        text = (
            "Ой, кажется, ты ещё не с нами ♥\n\n"
            "Подпишись на канал @yandex_market_gid и нажми кнопку "
            "«Проверить подписку» снова, чтобы участвовать в розыгрыше!"
        )
        bot.send_message(message.chat.id, text, reply_markup=markup)

@bot.callback_query_handler(func=lambda call: call.data == "check_subscription")
def check_subscription_callback(call: telebot.types.CallbackQuery):
    user_id = call.from_user.id

    if is_subscribed(user_id):
        # Обновляем сообщение на "успешное"
        text = (
            "Ура, ты с нами! ♥\n\n"
            "Ты участвуешь в розыгрыше сертификатов Яндекс Маркета:\n"
            "🥇 10 000 ₽\n"
            "🥈 5 000 ₽\n"
            "🥉 2 000 ₽\n\n"
            "Итоги 15 числа в канале. Удачи! 🌸"
        )
        bot.edit_message_text(
            chat_id=call.message.chat.id,
            message_id=call.message.message_id,
            text=text
        )
    else:
        # Показываем всплывающее уведомление, что подписка не оформлена
        bot.answer_callback_query(
            call.id,
            "Ты ещё не подписался на канал. Подпишись и попробуй снова.",
            show_alert=True
        )

if __name__ == "__main__":
    print("Бот запущен...")
    bot.infinity_polling()
