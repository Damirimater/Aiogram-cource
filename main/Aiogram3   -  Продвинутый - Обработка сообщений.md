# Создание своих фильтров

Не редко бывает, что обычные Command() или MagicFilter не подходят. Например, в случае когда нужно делать админ-панель. Тогда на помощь приходит класс Filter, с помощью которого можно создавать свои собственные кастомные фильтры.

```python
ADMINS = [12345, 67890]

class AdminProtect(Filter):
    def __init__(self):
        self.admins = ADMINS

    async def __call__(self, message: types.Message):
        return message.from_user.id in self.admins
```

Переменная ADMINS хранит в себе список администраторов бота в виде их уникальных Telegram ID.

Класс можно называть по-своему, в этом случае он назван AdminProtect, и является дочерним по отношению к классу Filter. Внутри класса используется конструктор __init__, который ничего не принимает, но создаёт поле self.admins, для дальнейшего его использования методами.

Метод __call__ по правилам ООП вызывается автоматически, в него передаётся сообщение - класс Message. Заметьте, чтобы работало с CallbackQuery, то нужно передать туда и его. Метод возвращает True если ID отправившего сообщение пользователя есть в списке admins.

Далее этот фильтр вызывается в декораторе также как и любой другой:

```python
@admin.message(AdminProtect(), Command('apanel'))
async def apanel(message: types.Message):
    await message.answer('Это панель администратора')
```

# Middlewares

aiogram предоставляет довольно мощный инструмент (такой же, как например в Django/FastAPI) для обработчиков, такой как middleware.

С английского, middlewares переводится как промежуточное программное обеспечение. В отличии от перечисленных фреймворков выше, в aiogram есть небольшое отличие. Миддлвари могут срабатывать как до фильтров, так и после.

Middleware - это функция, которая активируется при каждом событии, полученном от Telegram Bot API в обработчиках.

  
Представьте, что у вас есть обработчик с фильтром. И вы можете создать функцию, которая абсолютно всегда будет срабатывать, до фильтра или после (outer middleware/inner middleware). Это очень полезно, когда вы хотите, например, сделать статистику бота.

![Основы промежуточного программного обеспечения](https://docs.aiogram.dev/en/latest/_images/basics_middleware.png)

Сначала срабатывает outer middleware (например, здесь вы можете проверять есть ли доступ у пользователя к боту), далее срабатывает фильтр (например CommandStart()), и срабатывает innder middleware (внутренний), который, например, может подсчитывать количество посетителей или обращений к боту в день, далее выполняется сам хэндлер.

```python
from typing import Callable, Dict, Any, Awaitable
from aiogram import BaseMiddleware
from aiogram.types import TelegramObject

class ExampleMiddleware(BaseMiddleware):
    async def __call__(
        self,
        handler: Callable[[TelegramObject, Dict[str, Any]], Awaitable[Any]],
        event: TelegramObject,
        data: Dict[str, Any]
    ) -> Any:
        print("Действия до обработчика")
        result = await handler(event, data) # Запуск хэндлера
        print("Действия после обработчика")
        return result
```

Любой миддлварь должен содержать в себе метод __call__, принимающий в себя три параметра:  
**handler** - сам объект хэндлера (обработчика), который будет выполнен (или не выполнен при каком-то условии).  
**event** - тип Telegram-объекта, например Update, Message, CallbackQuery, InlineQuery. Если точно знаете какой апдейт хотите обработать, то вместо TelegramObject можно написать Message.  
**data** - связанные данные, например FSM. Можем добавлять туда какие-то свои данные.

Если вы не хотите обрабатывать хэндлер при каком-то условии, можно просто ничего не возвращать. Но если вы хотите обработать запрос, то обязательно указывайте await handler(event, data)
\


Продолжение - [[Aiogram3 -  Продвинутый 2]]
