# Контракт данных IN/OUT Парсера

## Внешние данные
**`book_info: dict`**

**`book_data: dict`**


## Вход

### Файл pdf
**Тип документа**: image+OCR (`tesseract`)
**Размер докумена**: 100+ станиц

### PDF-движок
**Название**: `mupdf`/`fitz`
- `c` язык, быстрее аналога - `pdfplumber`



### Входящие данные
**`book_info`**: данные о книге. Внешний. Вид:
```
{
'id': int,
'series': str | None,
'authors': str | None,
'name': str,
'pages': int,
}
```
**`pdf_doc`**: результат выполнения `fitz.open(book)`. Внутренний.

## Обработка и выход. Класс Парсера

### Инициализация
**Архитектура**: класс-накопитель состояния целого документа. Вид:
`Parser(pdf_doc, book_info)`

**`__init__`**: инициализация `pdf_doc`, `book_info`, `_construct` и `book_data`
- `_construct` - конструктор страницы документа вида. Внутренний. Вид:
```
{
'body': str
'page': list
}
```

- `book_data` - компилятор конструкторов всей книги. Внешний. Вид:
```
{
'id': int,
'book_id': int,
'data': [_construct]
}
```

### Обработка

#### `get_lines(self) -> list[dict]`

**Задача**: Внутренняя работа с линиями, используя `self.pdf_doc`

**Метод работы с документом**: `get_text('dict')`
- Нужна работа со `spans` и, впоследствии, фильтрация `fitz layout` (`y_tolerance`)
- Создает список `line_sample` вида:
```
{
'text': str,
'x0': int,
'y0': int,
'fontsize': int
}
```

**Фильтрация скрытых символов**: Через внутреннюю функцию `_filter_symbols(lines)`. Фильтрация невидимых обьектов типов:
`\xad`, `\n`, `\u200b`, `\xa0`, `\\`, `NFKD`-лигатуры (ае и пр.)

**Фильтрация верстки**: Фильтрация исходя из `heuristics.md` через внутреннюю функцию `_filter_layout(lines)`

**Вывод (`return`)**: 
```
[filtered_line_samples]
```

#### `execute(self) -> dict: book_data`

**Задача/Роль**: Исполнитель, вызываемый внешне. Компилятор.

**Ход работы**: Кладет результат работы `get_lines()` в `_construct`, 
и вкладывает `_construct` в `book_data`

**Вывод (`return`)**: `book_data`