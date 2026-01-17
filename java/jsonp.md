# Java: JSONP

- [jsonp](https://github.com/javaee/jsonp) by [javaee](https://github.com/javaee)
	- _Java API for JSON Processing (JSON-P)_

## Tokenizer

:robot_face::

```
Буфер: XXXssssssssssssXXXXXXXXXXXXXXXXXXXXXXrrrrrrrrrrrrrrXXXXXX
       ^           ^                     ^             ^
       |           |                     |             |
    storeBegin  storeEnd            readBegin      readEnd
        ↓             ↓                     ↓             ↓
     Начало       Конец              Следующий       Конец
     токена       токена             символ для    доступных
                                     чтения        данных

- `ssss...` — символы текущего токена (строка/число)
- `rrrr...` — непрочитанные символы из потока
- `X` — свободная/обработанная область
```

- :open_file_folder: `impl/src/main/java/org/glassfish/json/JsonTokenizer.java`
	- `readBegin`, `readEnd`: `int` — :robot_face: указатели для чтения из буфера
	- `storeBegin`, `storeEnd`: `int` - :robot_face: указатели для хранения значений токенов. Начало и конец текущего токена (строки, числа).
		- :robot_face:  Между `storeBegin` и `storeEnd`** — символы текущего токена (например, строки без кавычек или числа).
		- :robot_face: После парсинга токена методы `getValue()`, `getBigDecimal()` используют эти индексы для извлечения значения.
	- `bufferOffset`: `int` -  offset in the stream for the start of the buffer, will be used in calculating JsonLocation's stream offset,  bufferOffsetcolumn no.
		- :robot_face: При сдвиге данных в буфере (в `fillBuf()`) `bufferOffset` увеличивается на количество обработанных символов.
	- `lastLineOffset: int` -  offset of the last \r\n or \n. used to calculate column number // of a token or an error.
	- `nextToken(): JsonToken`
		- Здесь происходит посимвольный разбор
	- `readString()`
		- `storeBegin = storeEnd = readBegin`
	- `read()`
		- if `readBegin == readEnd` // need to fill the buffer`
			- `len = fillBuf()`
		- отличается от peek только тем, что происходит инкремент:
			- `return buf[readBegin++]`
	- `peek(): int` - :robot_face: "подглядеть" следующий символ без продвижения
		- if `readBegin == readEnd` // need to fill the buffer
			- `len = fillBuf()`
			- `readBegin = storeEnd;`
			- `readEnd = readBegin+len;`
	- `getBigDecimal()`
		- `new BigDecimal(buf, storeBegin, storeEnd-storeBegin);`

## JsonParser

- :open_file_folder: `impl/src/main/java/org/glassfish/json/JsonParserImpl.java`
	- `currentEvent`
	- `ObjectContext extends Context {`
		- `skip() {`
		- `token = tokenizer.nextToken();`
			- `while (!(token == JsonToken.CURLYCLOSE && depth == 0))`
	- `ArrayContext extends Context {`
		- `skip() {`
			- `token = tokenizer.nextToken();`
				- `} while (!(token == JsonToken.SQUARECLOSE && depth == 0));`
	- `final class JsonTokenizer implements Closeable {`

## How event is formed

- :symbols::mag_right: `KEY_NAME`

- :open_file_folder: `impl/src/main/java/org/glassfish/json/JsonParserImpl.java`
	- :package: `JsonParserImpl`
		- `currentEvent: Event`
		- `currentContext: Context = NoneContext()`
		- `next(): Event`
			- `currentEvent = currentContext.getNextEvent()`
	- :package: `NoneContext`
		- `extends Context `
		- `getNextEvent(): Event`
			- `JsonToken token = tokenizer.nextToken()`
			- if `token == JsonToken.CURLYOPEN`
				- `return Event.START_OBJECT`
			- if `token == JsonToken.SQUAREOPEN`
				- `return Event.START_ARRAY`
			- if `token.isValue()`
				- `return token.getEvent()`
					- :package: `enum JsonToken`
						- ` getEvent(): JsonParser.Event`
							- `return event`

## How token is read

- :open_file_folder: `impl/src/main/java/org/glassfish/json/JsonTokenizer.java`
	- :package: `JsonTokenizer`
		- `nextToken()`
			- case `t`
                - `readTrue();`
                - `return JsonToken.TRUE`
			- case `"`
				- `readString()`
				- `return JsonToken.STRING`
			- //  . . . . .
		- `readTrue()`
			- Expects `rue` chars consequently
		- `readString()`
			- `nPlace: boolean = true` // when inPlace is true, no need to copy chars
			- `do ... while(true)`
				- if `ch >= 0x20 && ch != 0x22 && ch != 0x5c`
					- if `!inPlace`
						- `buf[storeEnd] = (char)ch`
				- case `"`
					- return
				- case `\\`
					- `inPlace = false`  // Now onwards need to copy chars
					- `unescape()`
					- `break`