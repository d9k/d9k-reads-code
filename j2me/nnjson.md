# NNjson

- [shinovon/NNJSON@9179340](https://github.com/shinovon/NNJSON/tree/9179340491095032bbad67d5fb84a9211f39dfed)

## :package: `BufferedReader`

- :open_file_folder: `src/cc/nnproject/json/BufferedReader.java`

- `ensureBuf()`
	- `result = true`
	- if `iBufPos >= iBufAmount`
		- if `fillBuf() == -1`
			- `result = false`
	- `return result`

- `fillBuf()`
	- // Fills the buffer from the Reader and resets the buffer counters
	- `readCount = iReader.read(iBuf)`
	- Reset the buffer counters if reading succeeded.
	- `return readCount`

### :link: in :package: `JSONStream`

- :open_file_folder: `src/cc/nnproject/json/JSONStream.java`

- `init(InputStream in)`
	- if `buffer`
		- `reader = new BufferedReader(reader)`
- `next(): char`
	- `r: int = reader.read()`
- `skip(int n)`
	- `reader.skip(n)`
