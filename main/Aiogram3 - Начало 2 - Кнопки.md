## Клавиатура

урок - https://youtu.be/_l7dMZWbk-M

```python
from aiogram.types import ReplyKeyboardMarkup, KeyboardButton

main = ReplyKeyboardMarkup(keyboard=[
	[KeyboardButton(text=''),
	 KeyboardButton(text='отправить контакт:', request_contact=True)],
	 [KeyboardButton(text='')]
],
							resize_keyboard=True,
							input_field_placeholder='')
```

Здесь, переменная main является экземпляром объекта (клавиатурой) от ReplyKeyboardMarkup. Это класс, который создаёт клавиатуру.

Далее, внутри этого класса в инициализатор передаётся аргумент keyboard, который должен являться списком. Внутри этого списка создаются кнопки. 1 список = 1 ряд. В моем случае, внутри этого списка два ряда, так как там ещё два списка. Теперь эту клавиатуру нужно вызвать. Клавиатура может открываться только с отправкой какого-либо сообщения (апдейта). Для этого, при отправке команды /start, будем открывать эту клавиатуру.

## Inline клавиатура

```python
from aiogram.types import InlineKeyboardButton, InlineKeyboardMarkup
```

Создание Inline клавиатуры происходит очень похожим способом

```python
main_inline = InlineKeyboardMarkup(inline_keyboard=[
	[InlineKeyboardButton=(text='', url='https://archlinux.com')]
])
```

И подставляется в роутер:\

```python
@router.message(CommandStart())
async def cmd_start(message: Message):
    await message.answer(f'Привет!', reply_markup=kb.inline_main)
```

Отличия мы видим в классах, здесь используется InlineKeyboardMarkup, который принимает в себя аргумент inline_keyboard, и в него мы должны передать список, внутри которого находится список кнопок. Точно также, 1 список внутри = 1 ряд кнопок.

Ещё одно необычное поведение, это то, что при нажатии на кнопку я указал URL. Всё дело в том, что в отличии от Reply кнопок, Inline не могут отправлять сообщение в чат, поэтому они могут перебрасывать по ссылке, открывать WebApp или отправлять коллбэк. Что-то одно из этого должно быть указано обязательно.

## Понятие callback

Как и говорилось в прошлом уроке, Inline клавиатура может отправлять коллбэк (callback). В aiogram это некая строчка (как сообщение), которая с помощью фильтра вызывает нужную функцию (хэндлер). Дело в том, что этот коллбэк не видит пользователь, и мы можем передавать оттуда что-то полезное: например ID.

Давайте рассмотрим пример Inline клавиатуры с коллбэком:

```python
inline_main = InlineKeyboardMarkup(inline_keyboard=[
    [InlineKeyboardButton(text='Корзина', callback_data='basket')],
    [InlineKeyboardButton(text='Каталог', callback_data='catalog')],
    [InlineKeyboardButton(text='Контакты', callback_data='contacts')]
])
```

Коллбэки basket/catalog/contacts будут обрабатываться хэндлерами по примеру:

```python
from aiogram import Router, F
from aiogram.types import Message, CallbackQuery
from aiogram.filters import CommandStart

import app.keyboards as kb

router = Router()


@router.message(CommandStart())
async def cmd_start(message: Message):
    await message.answer(f'Привет!', reply_markup=kb.inline_main)


@router.callback_query(F.data == 'basket')
async def basket(callback: CallbackQuery):
    await callback.message.answer('Ваша корзина пуста.')
```

Прошу обратить внимание на импорт класса CallbackQuery. Далее, у роутера вызывается другой метод: callback_query, и раннее импортированный класс передаётся в качестве параметра в функцию. Мы отправляем сообщение пользователю с помощью метода callback.message.answer. Что видим в итоге:

После нажатия на кнопку Корзина, отправляется сообщение, но нажатая кнопка всё ещё светится. Дело в том, что на коллбэк мы должны ответить методом callback.answer:

```python
@router.callback_query(F.data == 'basket')
async def basket(callback: CallbackQuery):
    await callback.answer('Вы выбрали корзину.')
    await callback.message.answer('Ваша корзина пуста.')
```

В таком случае кнопка перестаёт светиться, и по середине выскакивает уведомление 

При отправке подобного уведомления мы можем вывести не исчезающее уведомление, а полноценное окно, добавив аргумент show_alert=True:

```python
@router.callback_query(F.data == 'basket')
async def basket(callback: CallbackQuery):
    await callback.answer('Вы выбрали корзину.', show_alert=True)
    await callback.message.answer('Ваша корзина пуста.')
```

### Заключение

Вы успешно научились создавать и использовать Reply и Inline клавиатуру: использовать классы ReplyKeyboardMarkup и InlineKeyboardMarkup, а также передавать списки: [ [], [] ].

Reply клавиатура появляется снизу, и отправляет содержимое кнопки в чат. Это сообщение можно ловить через фильтр F.text и обрабатывать нужным способом.

Inline клавиатура появляется под сообщением, может открывать ссылку, WebApp или отправлять коллбэк. Callback ловится с помощью фильтра F.data и обрабатывается методом роутера .callback_query.

Все клавиатуры обязательно должны крепиться к какому-либо сообщению с помощью метода reply_markup.

## ReplyKeyboardBuilder

урок - https://youtu.be/6eOmS6vDEww

Кроме обычного метода создания клавиатуры, в aiogram есть ещё один способ - через билдеры (Builder).

Представим, что у вас есть база данных, в которой хранится какая-то информация: например, о товарах. И вы получаете из БД список товаров:

```python
data = ("Nike", "Adidas", "Reebok")
```

И из этого кортежа вам нужно создать клавиатуру. Есть условие, что данные всегда могут быть разными. То есть вы можете добавлять или удалять товары из БД. Тогда нужно создать какую-то динамическую клавиатуру. Сделать это можно конечно и обычным способом:

```python
from aiogram.types import (ReplyKeyboardMarkup, KeyboardButton,
                           InlineKeyboardMarkup, InlineKeyboardButton)

data = ("Nike", "Adidas", "Reebok")

def brands():
    buttons = []
    for brand in data:
        buttons.append([KeyboardButton(text=brand)])
    keyboard = ReplyKeyboardMarkup(keyboard=buttons)
    return keyboard
```

В теле функции brands создаётся список, и в него добавляются ещё списки. Это не очень удобно, так как мы не можем контролировать количество кнопок в строчке.  
Поэтому был создан более удобный метод:

```python
from aiogram.types import KeyboardButton, InlineKeyboardButton
from aiogram.utils.keyboard import ReplyKeyboardBuilder, InlineKeyboardBuilder

data = ("Nike", "Adidas", "Reebok")

def brands():
    keyboard = ReplyKeyboardBuilder()
    for brand in data:
        keyboard.add(KeyboardButton(text=brand))
    return keyboard.adjust(2).as_markup()
```

Минуя работу со списками мы добавляем кнопки прямо в объект ReplyKeyboardBuilder. Метод add добавляет кнопки, а при её возврате используется метод adjust, с помощью которого регулируется количество кнопок в ряду. В самом конце обязательно указать as_markup().

![](https://sudoteach.com/media/uploads/2024/02/14/image.png)

Заметьте, что я создал новый файл **builder.py** в папке app.

В хэндлерах клавиатура указывается аналогичным способом:

![](https://sudoteach.com/media/uploads/2024/02/14/image_aoQGQrm.png)

И опять же, абсолютно аналогично всё работает с InlineKeyboardBuilder.

Продолжение - [[Aiogram3 - Начало 3 - Finite State Machine]]
