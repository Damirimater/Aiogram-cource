## FSM Context

Одна из основных задач Телеграм бота - общение с пользователем. Состояния используются для создания диалога между ботом и юзером, а именно: ветвление диалогов (в зависимости от ответов) и запоминание нужной информации.

Представим, что у нас есть процесс регистрации, и мы просим пользователя ввести имя, а потом отправить фотографию. Как мы поймем что пользователь прислал имя, ведь этот процесс никак не обработать с помощью кнопки или callback'а. Поэтому на помощь приходят состояния. Мы присваиваем пользователю определенное состояние, в котором он будет находиться. И уже в зависимости от этого состояния мы определяем что прислал нам юзер.

Ещё один пример из документации:

![FSM Example](https://docs.aiogram.dev/en/latest/_images/fsm_example.png)

Здесь используется всё: и ветвление, и запоминание информации о юзере. Каждый шаг - это отдельное состояние. Но как мы видим, в зависимости от ответа мы можем пропускать какие-то ходы. Нам же интереснее ещё и записывать итог в базу данных. Давайте рассмотрим пример с процессом регистрации. И для начала, нам нужно создать эти состояния, т.е в файле **handlers** класс с полями:

```python
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup

class Reg(StatesGroup):
    name = State()
    number = State()
    photo = State()
```

Представим, что мы собираем у пользователя следующую информацию: name (имя), number (номер телефона), photo (ава). Для этого мы создаём дочерний класс Reg по отношению к StatesGroup, и поля в нём, которые являются объектами класса State. Тем самым мы прописали состояния, т.е этапы, которые должен пройти пользователь.

Теперь пришло время их применить. Для этого, мы можем воспользоваться классом FSMContext, и его методами set_state и clear:

```python
@router.message(CommandStart())
async def cmd_start(message: Message, state: FSMContext):
    await state.set_state(Reg.name) # Установка состояния Reg.name
    await message.answer(f'Привет! Введит своё имя')
```

С помощью метода set_state было установлено состояние name: т.е процесс регистрации пошел, и мы просим ввести юзера своё имя. После того, как он напишет его и отправит, мы должны словить это имя по состоянию и тут же сохранить:

```python
@router.message(Reg.name)
async def reg_name(message: Message, state: FSMContext):
    await state.update_data(name=message.text)
    await state.set_state(Reg.number)
    await message.answer('Отправьте свой номер телефона')


@router.message(Reg.number)
async def reg_number(message: Message, state: FSMContext):
    await state.update_data(number=message.text)
    await state.set_state(Reg.photo)
    await message.answer('Отправьте фото')


@router.message(Reg.photo, F.photo)
async def reg_photo(message: Message, state: FSMContext):
    await state.update_data(photo=message.photo[-1].file_id)
    data = await state.get_data()
    await message.answer_photo(photo=data['photo'],
                               caption=f'Информация о Вас: {data['name']}, {data['number']}')
    await state.clear()
```

Вся информация, присланная пользователем сохраняется в метод state.update_data и ему даётся какой-либо ключ. Всё это превращается в словарь (dictionary).

В самом конце, используя метод get_data сохраненная информация была выведена через ключ-индекс, а состояние было очищено через метод clear.

Обязательно попробуйте данные методы самостоятельно.

Продолжение - [[Aiogram3   -  Продвинутый - Обработка сообщений]]