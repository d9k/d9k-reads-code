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
		- :robot_face:  Между `storeBegin` и `storeEnd` — символы текущего токена (например, строки без кавычек или числа).
		- :robot_face: После парсинга токена методы `getValue()`, `getBigDecimal()` используют эти индексы для извлечения значения.
	- `bufferOffset`: `int` -  offset in the stream for the start of the buffer, will be used in calculating JsonLocation's stream offset,  bufferOffsetcolumn no.
		- :robot_face: При сдвиге данных в буфере (в `fillBuf()`) `bufferOffset` увеличивается на количество обработанных символов.
	- `lastLineOffset: int` -  offset of the last \r\n or \n. used to calculate column number // of a token or an error.
	- `nextToken(): JsonToken`
		- Здесь происходит посимвольный разбор
	- `readString()`
		- `storeBegin = storeEnd = readBegin`
	- `read()`
		- if `readBegin == readEnd` // need to fill the buffer
			- `len = fillBuf()` // fill buffer and remember how much was read
			- `readBegin = storeEnd`
			- `readEnd = readBegin+len`
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

### :test_tube: Tests

- :open_file_folder: `tests/src/test/java/org/glassfish/json/tests/JsonParserTest.java`

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
			- `currentContext = new ArrayContext()`
		- if `token.isValue()`
			- `return token.getEvent()`
				- :package: `enum JsonToken`
					- ` getEvent(): JsonParser.Event`
						- `return event`

- :package: `ArrayContext`
	- `getNextEvent()`

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
			- `inPlace: boolean = true` // when inPlace is true, no need to copy chars
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

## ~~JsonArrayBuilderImpl~~

- :open_file_folder: ~~`impl/src/main/java/org/glassfish/json/JsonObjectBuilderImpl.java`~~

## :package: `BufferPoolImpl`

- :package: `JsonTokenizer`
	- :open_file_folder: `j2mePort/org/glassfish/json/JsonTokenizer.java`
	- `JsonTokenizer(Reader reader, BufferPool bufferPool)`
		- `buf = bufferPool.take();`
	- `fillBuf()`
		- // buffer is full, double the capacity
		- `doubleBuf: char[] = Arrays.copyOf(buf, 2 * buf.length);`
		- `bufferPool.recycle(buf)`
		- `buf = doubleBuf;`
		- // Fill the rest of the buf
		- `return reader.read(buf, storeEnd, buf.length-storeEnd);`


- :package: `BufferPoolImpl`
	- `queue`: `WeakReference< ConcurrentLinkedQueue<char[]> >`
	- :open_file_folder: `impl/src/main/java/org/glassfish/json/BufferPoolImpl.java`
	- implements:package: `BufferPool` from :open_file_folder: `impl/src/main/java/org/glassfish/json/api/BufferPool.java`
	- `recycle(t: char[])`
		- // Returns an object back to the pool.
		- `getQueue().offer(t);`
	- `take(): char[] `
		- // If no object is available in the pool, this method creates a new one.
		- `char[] t = getQueue().poll();`
		- if `!t`
			- `return new char[4096]`

- :package: `ConcurrentLinkedQueue<E>`
	- `poll()`
		- // Retrieves and removes the head of this queue, or returns null if this queue is empty.
	- `offer(e: E): boolean`
		- // Inserts the specified element into this queue if it is possible to do so immediately without violating capacity restrictions. Return true if the element was added to this queue
		- // Unlike `add()` doesn't throw exception if capacity was exceeded, just returns `false`

- :package: `Reader`
	- `read(char cbuf[], int off, int len)`
		- // Reads characters into a portion of an array.
		- // cbuf:  Destination buffer
		* // off:   Offset at which to start storing characters
		* // len:   Maximum number of characters to read

### :speaking_head_in_silhouette: Дипсик, Почему в `BufferPoolImpl` буфер для чтения из JSON-файла именно так реализован, именно такие методы ConcurrentLinkedQueue вызываются? В чём минусы, в чём плюсы? / 2026.01.27

:robot_face:: ConcurrentLinkedQueue + WeakReference — "самоочищающийся" пул. **WeakReference позволяет GC очистить пул**, если он долго не используется. При сборке мусора `queue.get()` вернёт `null`, создаст новую очередь. Защита от "утечек памяти" при редком использовании пула.

:robot_face:: poll() в take() — неблокирующее получение. **`poll()`** возвращает `null`, если очередь пуста.  **Не бросает исключений** (в отличие от `remove()`)

:robot_face:: Минусы: Не подходит для очень больших JSON-строк (> 4096 символов). Парсеру придётся расширять буфер вручную (что он и делает через `Arrays.copyOf`)

:robot_face:: WeakReference может быть слишком агрессивным. Пул может очищаться между вызовами `take()` и `recycle()`. Потеря буферов при активном использовании.
	- :speaking_head_in_silhouette:: Но ведь основная "жёсткая" ссылка в JsonTokenizer будет оставаться, следовательно, и WeakReference сохранится?
		- :robot_face:: WeakReference на очередь, а не на буферы

:speaking_head_in_silhouette:: Скажи, а, получается, в данной реализации `JsonTokenizer` `buf[]` вообще не забывает старое прочитанное? То есть, если на устройства 1 GB оперативы, а надо прочитать 2 GB, то такая реализация не сможет? Я немного запутался.

:robot_face:: Файл читается порциями через Reader.read(). В буфере хранится только текущая обрабатываемая часть. Старые данные перезаписываются новыми.

:robot_face:: **Память не растёт**: Постоянно 4 КБ в памяти, независимо от размера файла. Старые данные **вытесняются** новыми (как в кольцевом буфере)

:robot_face:: НО если в JSON есть строка длиной > 4096 символов, буфер будет расти. Теоретически может вырасти до 2 ГБ при одной гигантской строке! При длинных строках происходят копирования и аллокации. Это может быть неэффективно для очень больших строк.

:robot_face:: Так что **1 ГБ ОЗУ хватит для парсинга 2 ГБ JSON**, если в нём нет отдельных строк по 1 ГБ каждая! Парсер будет читать файл кусочками, как видеоплеер читает длинное видео.