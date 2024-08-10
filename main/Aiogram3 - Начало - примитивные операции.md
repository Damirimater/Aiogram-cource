курс - https://sudoteach.com/content
### Установка

###### Windows: 
```
python -m venv .venv -  создает виртуальное окружение
```
```
.venv\Scripts\activate - активирует виртуальное окружение
```
```
pip install aiogram - устанавливает  aiogram
```

#### Linux:
```
python -m venv .venv
source .venv/bin/activate
pip install aiogram
```


### Что такое Aiogram
  Урок - https://youtu.be/lHMp7zwfu5Y
 ![[Pasted image 20240714193534.png]]
![[Pasted image 20240714194015.png]]


```python
import asyncio

from aiogram import Bot, Dispatcher


bot = Bot(token='') # Bot принимает в себя токен раннее созданного бота, # и инициализирует подключение к нему, мы можем создать несколько таких объектов # и подключить их всех к диспетчеру
dp = Dispatcher() #- это основной роутер, работа происходит через него, либо в  # него передают остальные роутеры. Его задача - обрабатывать входящие
# обновления: #сообщения, коллбэки, и так далее.

# Эта асинхронная(!) функция main() является основной функцией программы, и той # самой точкой входа. Внутри, обращаясь к диспетчеру, вызывается его метод  
# start_polling, в который передаётся объект класса Bot.
async def main():
    await dp.start_polling(bot)#Что это такое (polling)? Бот, то есть aiogram, а # именно эта функция, будет отправлять запрос на сервера телеграма, и если ответ # есть, телеграм даст ответ и наш бот его обработает, а если ответа нет, то
# функция просто будет дальше ожидать ответа от телеграма.


if __name__ == '__main__': # запускает функцию main(). asyncio.run() 
# используется здесь потому что функция main() является асинхронной
    asyncio.run(main())
```

### Обработка старта

```python
from aiogram.types import Message
from aiogram.filters import CommandStart


@dp.message(CommandStart())
async def cmd_start(message: Message):
	await message.reply('Hello')
```

Теперь бот отвечает нам

По скольку когда мы завершаем работу бота вылезает ошибка, нам надо сделать обработчик ошибок
```python
try:
	asyncio.run(main())
except:
	print('exit')
```

 Чтобы обезопаситься, надо вынести токен бота в отдельную папку .env с помощью библиотеки dotenv:

```python
TOKEN = 
```

```python
import os
from dotenv import load_dotenv

load_dotenv() #прогружает переменные в dotenv(.env)

bot = Bot(token=os.getenv('TOKEN))
```

### Установка dotenv
```
python3 -m venv venv
echo "aiogram<4.0" > requirements.txt
echo "pydantic-settings" >> requirements.txt
source venv/bin/activate
pip install -r requirements.txt
```

```
deactivate
```

Чтобы файл .env не был импортирован в git надо добавить файл .gitignore и написать в него имя файла который не надо импортировать(.env) 

Так же чтобы писать свои команды надо прописать обработчик команд:

```python
from aiogram.filters.command import Command

@dp.message(Command('cmd'))
```

Кроме метода message.answer, существует много других. Например, метод .reply:

```python
@dp.message(CommandStart())
async def cmd_start(message: Message):
    await message.reply('Привет!')
    await message.answer('Как дела?')
```

У reply и answer есть дополнения, например в виде отправки картинок:

```python
@dp.message(CommandStart())
async def cmd_start(message: Message):
    await message.reply('Привет!')
    await message.answer_photo(photo='https://sudoteach.com/static/assets/img/aiogram-banner.jpg', caption='Лучший курс по aiogram!')
``` 

Данные о пользователе и о сообщении автоматически отправляются в переменную message

### Логирование

Логирование - это важный момент в процессе разработки. Благодаря выводу в терминал информации о происходящем в программе вы сможете найти ошибки или понять почему бот работает тем или иным образом.

```python
import logging

if __name__ == '__main__':
	logging.basicConfig(level=logging.INFO)
	...
```

с помощью MagicFilters можно ловить обычный текст:

```python
from aiogram import F

@dp.message(F.text == 'hi')

@dp.message(F.photo)
```

Основная задача хэндлеров (обработчиков) - ловить и обрабатывать сообщения. Давайте разберёмся, как ловить от пользователя различные сообщения. Для этого импортируем модуль MagicFilter (класс F), и воспользуемся его методами:

|Фильтр|Описание|
|---|---|
|F.text == '123'|Ловит определённый текст|
|F.text.startswith('smth_')|Ловит определенный текст, который начинается с smth_|
|F.data == '123'|Ловит определенный Callback|
|F.data.startswith('smth_')|Ловит определенный Callback который начинается с smth_|
|F.photo|Ловит только фото (все)|
|F.document|Ловит только документы|
|F.from_user.id == 123|Ловит только от определенного юзера по его ID|
### Command()

Класс Command() предназначен для работы с командами, например /help (команда всегда начинается со слэша).

### CommandObject()

принимает аргумент после написания команды

```python
@dp.message(Command('get'))
async def cmd_get(message: Message, command: CommandObject):
    await message.answer(f'Вы ввели команду get с аргументом {command.args}')
```

Можно принимать и несколько. Тогда их нужно разделить между собой и перебрать:

```python
@dp.message(Command('get'))
async def cmd_get(message: Message, command: CommandObject):
    value1, value2 = command.args.split(' ', maxsplit=1)
    await message.answer(f'Вы ввели команду get с аргументом {value1} {value2}')
```

В этой конструкции возможны недоработки. Давайте рассмотрим все возмжные варианты ошибок:

```python
@dp.message(Command('get'))
async def cmd_get(message: Message, command: CommandObject):
    if not command.args:
        await message.answer('Аргументы не переданы')
        return
    try:
        value1, value2 = command.args.split(' ', maxsplit=1)
        await message.answer(f'Вы ввели команду get с аргументом {value1} {value2}')
    except:
        await message.answer('Были введены неправильные аргументы')
```

CommandStart() его задача - ловить команду /start. Но на самом деле не всё так просто. С помощью неё можно ловить дополнительные параметры, и делать например рефферальную систему: t.me/bot?start=123. Тогда бот получит сообщение в формате /start 123. Это называется deep links. Здесь может быть передан только 1 аргумент!

```python
@dp.message(CommandStart(deep_link=True))
async def cmd_start(message: Message, command: CommandObject):
    await message.answer(f'Привет! Ты пришел от {command.args}')
```

В этом случае команда принимает всё что угодно в ссылке. Давайте также рассмотрим все возможные варианты и допустим только передачу числа:

```python
@dp.message(CommandStart(deep_link=True))
async def cmd_start(message: Message, command: CommandObject):
    if command.args.isdigit():
        await message.answer(f'Привет! Ты пришел от {command.args}')
    else:
        await message.answer('Ошибка')
```

Или если вы НЕ хотите выводить сообщение об ошибке, то можно воспользоваться аргументом magic:

```python
@dp.message(CommandStart(deep_link=True, magic=F.args.isdigit()))
async def cmd_start(message: Message, command: CommandObject):
    await message.answer(f'Привет! Ты пришел от {command.args}')
```


### Роутеры 

урок - https://youtu.be/9OLCdHTDEZI

хэндлеры лучше перенести в отдельный файл handlers.py
также токен, диспетчер и load_dotenv() желательно перенести в функцию main()  
Импорты команд тоже надо перенести в handlers.py и импортировать туда фильтры

ипортируем в хэндлеры роутер и делает его экземпляр класса
с помощью  ctrl + F    можно заменить все  dp на router   

импортирует роутеры в основной файл и прописываем 
```python
dp.include_router(router)
```
чтобы маршрутизировать роутер в диспетчер и делить хэндлеры


#### Что произошло?

Обработчики из основного файла были перенесены в файл handlers. Это нужно для того, чтобы структурировать проект и отделить друг от друга разные функциональные части программы.  
Далее, в файле handlers был создан экземпляр класса Router, и передали его в Dispatcher (через include_router). Благодаря роутеру мы можем переносить обработчики в другие файлы, и просто передавать эти роутеры диспетчеру.

Теперь, файл run является точкой входа. Это основной скрипт, который запускает все остальные.

### Как это работает?

Например, диспетчер имеет 2 роутера, а последний роутер имеет ещё один дополнительный роутер:

![Nested routers example](https://docs.aiogram.dev/en/latest/_images/nested_routers_example.png)

В этом случае, процесс обработки обновления будет работать следующим образом:

![Nested routers example](https://docs.aiogram.dev/en/latest/_images/update_propagation_flow.png)

Из этого вывод, что роутер - это просто аналог диспетчера. Вся проблема в том, что мы не можем просто импортировать диспетчер в другой файл, поэтому пользуемся роутером.

Пример из документации: [https://docs.aiogram.dev/en/latest/dispatcher/finite_state_machine/index.html](https://docs.aiogram.dev/en/latest/dispatcher/finite_state_machine/index.html)

### ChatAction

 ChatAction нужен чтобы делать промежуточное состояние, например: (печатает..., Выбирает стикер...)

импортируем из aiogram.enums ChatAction и пишем 

```python
import asyncio
from aiogram.enums import ChatAction
...																			await message.bot.send_chat_action(chat_id=message.from_user.id,
								action=ChatAction.UPLOAD_PHOTO)#
await asyncio.sleep(2) #асинхронная задержка времени
```

Заметьте, что отправка происходит через класс message и его метод bot, где обязательно нужно указывать chat_id, он обычно равняется ID пользователя.

### Группы/каналы/группы/топики

Для работы с топиками форума или с группами, нет необычных действий. Достаточно сделать обычный обработчик и добавить бота в группу или форум. Однако, бывают моменты, когда нам нужно отправить сообщение в определенную группу и в определенный топик, тогда можно воспользоваться методом объекта message send_message и указать дополнительные параметры:

```python
@router.message(Command('test'))
async def cmd_test(message: Message):
    await message.bot.send_message(chat_id=message.chat.id,
                                   message_thread_id=message.message_thread_id,
                                   text='OK')
```

Заметьте, что в chat_id мы теперь передаем не ID пользователя, а группы. А message_thread_id передается в том случае, если это форум.

Продолжение - [[Aiogram3 - Начало 2 - Кнопки]]
