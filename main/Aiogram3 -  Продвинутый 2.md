# Как скачать файл?

## Скачивание файла вручную

Сначала, вы должны получить file_id файла, который вы хотите скачать. Информация о файлах отправляется боту и содержится в классе Message.

Например, скачать файл, который пришел боту:

```python
file_id = message.document.file_id
photo_id = message.photo[-1].file_id
```

Далее, используйте метод getFile, чтобы получить file_path:

```python
file = await bot.get_file(file_id)
file_path = file.file_path
```

После этого, используйте метод download_file от объекта бота.

```python
await bot.download_file(file_path, 'text.txt')
```

## Скачивание более легким способом

Скачивать файлы, находя file_id нужно не всегда. В таком случае, вы можете скачивать объект сразу:

```python
document = message.document
await bot.download(document)

#или

await bot.download(message.document)
```


# Ошибки

## Обработка ошибок

Стандартный вид обработки ошибок это использовать в хэндлерах блок try-except, однако в некоторых случаях вы можете использовать обработчики ошибок в роутерах или диспетчере.

Если использовать обработчик ошибок по отношению к диспетчеру, то он будет касаться всего бота. Если по отношению к роутеру, то только к этому роутеру.

```
from aiogram.types import ErrorEvent

@router.error()
async def error_handler(event: ErrorEvent):
    logger.critical("Критическая ошибка: %s", event.exception, exc_info=True)
    # сделать что-то с ошибкой
    ...
```

## Типы ошибок

TelegramForbiddenError - исключение, вызванное, когда бот кикнут с чата;  
TelegramUnauthorizedError - исключение, вызванное, когда токен бота указан неверно;  
TelegramConflictError - исключение, вызванное, когда токен бота уже используется другим приложением;  
TelegramNotFound - исключение, вызванное, когда чат, сообщение, пользователь и т.д не найдены;  
TelegramBadRequest - исключение, вызванное, когда запрос неверный;  
TelegramRetryAfter - исключение, вызванное, когда сработал флуд-контроль;  
TelegramNetworkError - стандартное исключение для всех сетевых ошибок Telegram

## Компилятор PyPy

В официальной документации aiogram сказано, что он поддерживает PyPy.

PyPy — это альтернативная реализация языка программирования Python, разработанная для повышения скорости выполнения Python-программ и улучшения их производительности.

Документация: [https://www.pypy.org/](https://www.pypy.org/)

Видео с установкой интерпретатора и запуском бота на нём:
https://www.youtube.com/watch?v=dZl3ZyXFTT4

  Продолжение - [[Aiogram3 - Продвинутый 3 - Проект]]
  
