from aiogram import Bot, Dispatcher, types
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.utils import executor

API_TOKEN = "YOUR_BOT_TOKEN_HERE"
ADMINS = [5848727463, 8150652959]

bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot)

CHANNELS = {
    "Send to Channel 1": -1002504723776,
    "Send to Channel 2": -1002489624380,
    "Send to Channel 3": -1002644325820,
    "Send to All": "ALL"
}

@dp.message_handler(content_types=types.ContentType.ANY)
async def forward_handler(message: types.Message):
    if message.from_user.id not in ADMINS:
        return

    kb = InlineKeyboardMarkup(row_width=2)
    for name in CHANNELS:
        kb.add(InlineKeyboardButton(text=name, callback_data=name))
    
    await message.reply("Choose where to send:", reply_markup=kb)

@dp.callback_query_handler()
async def callback_forward(call: types.CallbackQuery):
    msg = call.message.reply_to_message
    target = call.data

    if CHANNELS[target] == "ALL":
        for cid in CHANNELS.values():
            if cid != "ALL":
                await bot.copy_message(cid, msg.chat.id, msg.message_id)
    else:
        await bot.copy_message(CHANNELS[target], msg.chat.id, msg.message_id)

    await call.answer("Message forwarded!")

if __name__ == '__main__':
    executor.start_polling(dp)
