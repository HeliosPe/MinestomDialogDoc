## Структура диалогов

---
<details>
<summary>Оглавление</summary>

- [Вступление](../README.md#вступление)
- [Структура диалогов](STRUCTURE.md#структура-диалогов)
- [Типы диалогов](STRUCTURE.md#типы-диалогов)
</details>

- [DialogMetadata](STRUCTURE.md#dialogmetadata)
- [Notice](STRUCTURE.md#notice)
- [Confirmation](STRUCTURE.md#confirmation)
- [Multi Action]
- [Server Links]
- [Dialog List]

## DialogMetadata

---

Расмотрим общую структуру диалогов которую использует ванильный minecraft. Полезно для понимания реализации от Minestom
```js
//Подставлены значения по умолчанию где они имеются
{
"type": "...", //тип диалогового окна
"title": "...", //заголовок окна
"external_title": "...", //имя по которому можно найти извне. По умолчанию берёт значение title
"body": [...], //содержимое диалога. Либо текстовое сообщение, либо предметы из игры
"inputs": [...], //элементы для ввода (крутилки, кнопочки, поля для ввода)
"can_close_with_escape": true, //можно ли закрыть клавишей Escape
"pause": true, //Ставится ли игра на паузу при открытии диалога. Имеет смысл только для одиночной игры
"after_action": "close", //операция после нажатия кнопки или любого ввода от игрока.
    
"прочее зависит от type ": " . . . "
}
```
Это основная часть любого диалога в майнкрафт. Дополнительный поля зависят от типа диалога указанный в `type`. 
В Minestom общие поля для разных типов диалога упаковываются в `DialogMetadata` чей конструктор имеет следующую сигнатуру:
```java
DialogMetadata(
        Component title, 
        @Nullable Component externalTitle,
        boolean canCloseWithEscape, 
        boolean pause, 
        DialogAfterAction afterAction, 
        List<DialogBody> body, 
        List<DialogInput> inputs)
```
Перегрузкок у конструтора нет, потому надо вводить все поля, даже если предусмотрены значения по умолчанию. 

- `title` предоставляется методом ``Component.text(String text)``,
- `canCloseWithEscape` и `pause` обычные boolean,
- `afterAction` принимает одно из значений enum `DialogAfterAction`: `CLOSE`, `NONE`, `WAIT_FOR_RESPONSE` 
- `body` список объектов типа `DialogBody`. Подробнее про него написано [тут](COMPONENTS.md#dialogbody). Можно
подставить пустой список при помощи `List.of()`
- `inputs` список объектов типа `DialogInput`. Подробнее про него написано [тут](COMPONENTS.md#dialogbody). Можно
  подставить пустой список при помощи `List.of()`

Вот рабочий пример построения объекта типа `DialogMetadata`:
```java
new DialogMetadata(
        Component.text("Title"),
        null, false, false,
        DialogAfterAction.CLOSE,
        List.of(
                new DialogBody.PlainMessage(Component.text("Текст"), 200),
                new DialogBody.Item(ItemStack.of(Material.PANDA_SPAWN_EGG), null, false, false, 16, 16),
                new DialogBody.PlainMessage(Component.text("Ещё текст"), 100)
        ),
        List.of(
                new DialogInput.Text("id", 400, Component.text("label"), true, "Текст по умолчанию", 100, new DialogInput.Text.Multiline(10, 50))
        )
)
```
## Типы диалогов

---
Тип диалога задаёт конечную структуру. Поля json для каждого типа диалога можно изучить в [wiki](https://minecraft.wiki/w/Dialog). Здесь же будут 
сразу рассматриваться конструкторы интересующих нас объектов, так как они верно повторяют json.

Все данные типы диалогов от Minestom являются наследниками запечатанного интерфейса `Dialog`. Это означает что Minestom
не предусматривает создание новых типов диалогов. Однако существующие типы и поля в конструкторе 
`DialogMetadata` `body` и `inputs` предоставляют достаточно свободы для кастомизации.
## Notice

---
`Уведомление` это диалоговое окно с одной кнопкой действия в нижней части окна. Реализовывается вложенным классом 
`Dialog.Notice` чей конструктор имеет следующую сигнатуру:
```java
Notice(DialogMetadata metadata, DialogActionButton action)
```
- `metadata` хранит общие поля для всех типов диалога. Подробнее [тут](STRUCTURE.md#dialogmetadata),
- `action` описывают кнопку в нижней части окна. Подробнее [тут](COMPONENTS.md#dialogactionbutton)

Пример готового `Notice` диалога:
```java
var noticeDialog = new Dialog.Notice(
        new DialogMetadata(
                Component.text("Notice Dialog Tipe"),
                null,
                false,
                false,
                DialogAfterAction.CLOSE,
                List.of(
                        new DialogBody.PlainMessage(Component.text("Суть показать нечто и одну кнопку, которые обычно называют \"Ок\" или \"Подтверждаю\""), 200),
                        new DialogBody.Item(
                                ItemStack.of(Material.PANDA_SPAWN_EGG),
                                null, false, false, 16, 16),
                        new DialogBody.PlainMessage(Component.text("Загадка от Жака Фреско. На размылшение даётя 30 секунд."), 100)
                ),
                List.of()
        ),
        new DialogActionButton(Component.text("Ладно."), null, 100, null)
);
```
Как это выглядит:
![noticeDialogExample](../images/noticeDialogExample.png)

## Confirmation

---
`Подтверждение` это диалоговое окно с двумя кнопками действия в нижней части окна. Реализовывается вложенным классом
`Dialog.Confirmation` чей конструктор имеет следующую сигнатуру:
```java
Confirmation(DialogMetadata metadata, DialogActionButton yesButton, DialogActionButton noButton)
```
- `metadata` хранит общие поля для всех типов диалога. Подробнее [тут](STRUCTURE.md#dialogmetadata),
- `yesButton` описывают первую кнопку в нижней части окна.
- `noButton` описывают вторую кнопку в нижней части окна. Подробнее про `DialogActionButton` [тут](COMPONENTS.md#dialogactionbutton)

Пример готового `Confirmation` диалога:
```java
var confirmationDialog = new Dialog.Confirmation(
        new DialogMetadata(
                Component.text("Confirmation Dialog Tipe"),
                null,
                false,
                false,
                DialogAfterAction.CLOSE,
                List.of(
                        new DialogBody.PlainMessage(Component.text("Суть показать нечто и дать две кнопки, которые обычно называют \"Да\" и \"Нет\""), 200),
                        new DialogBody.Item(
                                ItemStack.of(Material.FLOW_BANNER_PATTERN),
                                null, false, false, 16, 16),
                        new DialogBody.PlainMessage(Component.text("Можно ли в тебя вставить?"), 100)
                ),
                List.of()
        ),
        new DialogActionButton(Component.text("Да."), null, 100, null),
        new DialogActionButton(Component.text("Нет."), null, 100, null)
);
```
Как это выглядит:
![noticeDialogExample](../images/confirmationDialogExample.png)

## Multi Action

---
## Server Links

---
## Dialog List

---


