import telebot
import requests

class APIException(Exception):
    pass

class CurrencyConverter:
    @staticmethod
    def get_price(base, quote, amount):
        response = requests.get("http://www.cbr.ru/scripts/XML_daily.asp")

        if response.status_code != 200:
            raise APIException("Ошибка при получении данных от ЦБ России")

        content = response.content.decode('windows-1251')  # Декодирование содержимого в кодировке windows-1251
        lines = content.split('<Valute ')
        rates = {}

        for line in lines[1:]:
            char_code = line.split('<CharCode>')[1].split('</CharCode>')[0]
            value = float(line.split('<Value>')[1].split('</Value>')[0].replace(',', '.'))
            rates[char_code] = value

        if base not in rates or quote not in rates:
            raise APIException("Неправильные коды валют")

        result = (rates[quote] / rates[base]) * float(amount)
        return round(result, 2)

# Токен вашего бота
bot_token = '6966456706:AAGobls22UYRz9kquxg8JO6_PVOrly0ySUU'
bot = telebot.TeleBot(bot_token)

@bot.message_handler(commands=['start', 'help'])
def handle_start_help(message):
    instructions = "Привет! Этот бот позволяет узнать цену на определённое количество валюты. " \
                   "Для получения цены отправьте сообщение в формате:\n" \
                   "<имя валюты, цену которой вы хотите узнать> <имя валюты, в которой надо узнать цену первой валюты> <количество первой валюты>.\n" \
                   "Например: USD EUR 100"
    bot.send_message(message.chat.id, instructions)

@bot.message_handler(commands=['values'])
def handle_values(message):
    currency_info = "Доступные валюты: USD, EUR, RUB и т.д."  # Замените на реальную информацию
    bot.send_message(message.chat.id, currency_info)

@bot.message_handler(func=lambda message: True)
def handle_message(message):
    try:
        text = message.text.upper().split()
        if len(text) != 3:
            raise APIException("Неправильный формат сообщения. Используйте <валюта1> <валюта2> <сумма>")

        base_currency, quote_currency, amount = text
        result = CurrencyConverter.get_price(base_currency, quote_currency, amount)
        response = f"Цена {amount} {base_currency} в {quote_currency}: {result}"
        bot.send_message(message.chat.id, response)

    except APIException as e:
        bot.send_message(message.chat.id, f"Ошибка: {str(e)}")

bot.polling()


# имя бота в телеграмм https://t.me/akkosbot
