# Technical Cheat Sheet: CleverReader Output Data Formatter Syntax
*Reference: Data_formatter.pdf*

This cheat sheet serves as a structured, machine-readable reference for translating plain-English formatting requests into exact data formatter command strings for the scanner.

---

## 1. Syntax Rules & Token Formats

Data Formatter commands manipulate a **virtual cursor** along the scanned barcode data string to select, search, navigate, modify, and format output.

| Token | Data Type | Representation & Constraints | Example |
| :--- | :--- | :--- | :--- |
| **`CMD`** | 2-char ID | The capitalized hex representation of the command code. | `F1`, `F2`, `EA`, `BA` |
| **`xx`, `yy`, `ss`** | 2-digit Hex | Hexadecimal representation of a single ASCII character byte. | `0D` (CR), `20` (Space), `30` (0) |
| **`nn`** (Count) | 2-digit Dec | Decimal count representing quantities or positions (`00`–`99`). | `05` (5 characters), `15` (15 keys) |
| **`nnnn`** (Count) | 4-digit Dec | Decimal count for larger offsets, lengths, or delays (`0000`–`9999`). | `0200` (200), `0003` (3) |
| **`S`** (String) | Hex String | Continuous stream of hex values representing a text string. | `54657374` ("Test") |

---

## 2. Comprehensive Command Registry

### Send Commands (Output Generation)

*   **`F1xx`** — **Send All Characters**
    *   *Description:* Sends all characters from the current cursor position to the end of the barcode, followed by the insert character `xx`.
    *   *Syntax:* `F1xx` (where `xx` is the trailing insert character's hex value; 00 is End-of-string filler, no-op separator).

*   **`F2nnxx`** — **Send a Number of Characters**
    *   *Description:* Sends `nn` characters starting from the current cursor position, followed by the insert character `xx`. If the end of the data is reached before `nn` characters, it stops sending data and inserts `xx`.
    *   *Syntax:* `F2nnxx` (`nn` = decimal 00-99, `xx` = insert character hex; 00 is End-of-string filler, no-op separator and the cursor stops here).
	* 	*Example:* Send first 10 characters, followed by a carriage return.
	*   *F2100D*
	*   *F2*: “Send a number of characters” command
	*   *10*: Send 10 charcters
	*   *0D*: Carriage return hex value
	*   *Input*: 1234567890ABCDEFGHIJ
	*   *Output*: 1234567890<CR>

*   **`F3ssxx`** — **Send All Characters Up to a Character**
    *   *Description:* Sends all characters starting from the current cursor position up to (but not including) the target character `ss`. Follows with insert character `xx`. **Moves the cursor to `ss`**.
    *   *Syntax:* `F3ssxx` (`ss` = target character hex, `xx` = insert character hex; 00 is End-of-string filler, no-op separator).
	* 	*Example:* Send all characters up to but not including “D,” followed by a carriage return
	*	*F3440D*
	*	*F3*: “Send all characters up to a particular character” command
	*	*44*: ASCII for 'D'
	*   *0D*: Carriage return hex value
	*   *Input*: 1234567890ABCDEFGHIJ
	*   *Output*: 1234567890ABC<CR>

*   **`E9nn`** — **Send All But the Last Characters**
    *   *Description:* Sends all characters starting from the current cursor position except for the last `nn` characters of the barcode. **Moves the cursor to 1 position past the last sent character**.
    *   *Syntax:* `E9nn` (`nn` = decimal 00-99).

*   **`F4xxnn`** — **Insert a Character Multiple Times**
    *   *Description:* Inserts character `xx` into the output message `nn` times. **Does not move the cursor**.
    *   *Syntax:* `F4xxnn` (`xx` = character hex, `nn` = decimal repetitions 00-99).
	*	*Example*: Send all characters except for the last 8, followed by 2 tabs.
	*	*E908F40902*
	*	*E9*: “Send all but the last characters” command
	*	*08*: number of characters at the end to ignore
	*	*F4*: “Insert a character multiple times” command
	*	*09*: hex value for a horizontal tab
	*	*02*: number of time the tab character is sent
	*   *Input*: 1234567890ABCDEFGHIJ
	*   *Output*: 1234567890AB<TAB><TAB>
	
*   **`B3`** — **Insert Symbology Name**
    *   *Description:* Inserts the name of the scanned barcode's symbology (e.g., "Code128") without moving the cursor.
    *   *Syntax:* `B3`

*   **`B4`** — **Insert Barcode Length**
    *   *Description:* Inserts the numeric length of the scanned barcode as a string (omitting leading zeros) without moving the cursor.
    *   *Syntax:* `B4`
	*	*Example*: Send the symbology name and length before the barcode data from the barcode above. Break up these insertions with spaces. End with a carriage return.
	*	*B3F42001B4F42001F10D*
	*	*B3*: “Insert Symbology Name” command
	*	*F4*: “Insert a character multiple times” command
	*	*20*: hex value for a space
	*	*01*: number of time the space character is sent
	*	*F1*: "Send All Characters" command
	*	*0D*: Carriage return hex value
	*   *Input*: 1234567890ABCDEFGHIJ
	*   *Output*: Code128 20 1234567890ABCDEFGHIJ<CR>

---

### Cursor Movement Commands

*   **`F5nn`** — **Move Cursor Forward**
    *   *Description:* Advances the virtual cursor ahead by `nn` characters.
    *   *Syntax:* `F5nn` (`nn` = decimal offset 00-99).
	*	*Example*: Move the cursor forward 3 characters, then send the rest of the barcode data from the barcode above. End with a carriage return.
	*	*F503F10D*
	*	*F5*: “Move the cursor forward a number of characters” command
	*	*03*: number of characters to move the cursor
	*	*F1*: "Send All Characters" command
	*	*0D*: Carriage return hex value
	*   *Input*: 1234567890ABCDEFGHIJ
	*   *Output*: 4567890ABCDEFGHIJ<CR>

*   **`F6nn`** — **Move Cursor Backward**
    *   *Description:* Rewinds the virtual cursor back by `nn` characters.
    *   *Syntax:* `F6nn` (`nn` = decimal offset 00-99). Note that the nn=(number of backward cursor moves - 1). Hence to move back 1 place, nn=0
	*	*Example*: Move the cursor to End, and move cursor backward by 1
	*	*EAF600*
	*	*EA*: Move cursor to the end
	*	*F6*: Move cursor backward
	*	*00*: 1 place

*   **`F7`** — **Move Cursor to Beginning**
    *   *Description:* Resets the virtual cursor to the first character of the scanned data.
    *   *Syntax:* `F7`

*   **`EA`** — **Move Cursor to End**
    *   *Description:* Positions the virtual cursor at the very last character of the scanned data.
    *   *Syntax:* `EA`

---

### Search Commands

*   **`F8xx`** — **Search Forward for a Character**
    *   *Description:* Searches forward from the current position for character `xx`. **Leaves the cursor pointing to `xx`**.
    *   *Syntax:* `F8xx` (`xx` = target character hex).
	*	*Example*: Search for the letter “D” in barcodes and send all the data that follows, including the “D”
	*	*F844F10D*
	*	*F8*: “Search forward for a character” command
	*	*44*: ASCII code for 'D'
	*	*F1*: "Send All Characters" command
	*	*0D*: Carriage return hex value
	*   *Input*: 1234567890ABCDEFGHIJ
	*   *Output*: DEFGHIJ<CR>

*   **`F9xx`** — **Search Backward for a Character**
    *   *Description:* Searches backward from the current position for character `xx`. **Leaves the cursor pointing to `xx`**.
    *   *Syntax:* `F9xx` (`xx` = target character hex).

*   **`B0nnnnS`** — **Search Forward for a String**
    *   *Description:* Searches forward from the current position for string `S` of length `nnnn`. **Leaves the cursor pointing to the beginning of string `S`**.
    *   *Syntax:* `B0nnnnS` (`nnnn` = 4-digit decimal length, `S` = ASCII hex bytes of string).
	*	*Example*: Search for the letters “FGH” in barcodes and send all the data that follows, including “FGH”
	*	*B00003464748F10D*
	*	*B0*: “Search forward for a string” command
	*	*0003*: Length of string = 3 characters
	*	*46*: ASCII code for 'F'
	*	*47*: ASCII code for 'G'
	*	*48*: ASCII code for 'H'
	*	*F1*: "Send All Characters" command
	*	*0D*: Carriage return hex value
	*   *Input*: 1234567890ABCDEFGHIJ
	*   *Output*: FGHIJ<CR>

*   **`B1nnnnS`** — **Search Backward for a String**
    *   *Description:* Searches backward from the current position for string `S` of length `nnnn`. **Leaves the cursor pointing to the beginning of string `S`**.
    *   *Syntax:* `B1nnnnS` (`nnnn` = 4-digit decimal length, `S` = ASCII hex bytes of string).

*   **`E6xx`** — **Search Forward for Non-Matching Character**
    *   *Description:* Searches forward for the first character that is **not** `xx`. **Leaves the cursor pointing to the non-`xx` character**.
    *   *Syntax:* `E6xx` (`xx` = character hex to ignore).
	*	*Example*: Remove zeros at the beginning of barcode data
	*	*E630F10D*
	*	*E6*: “Search forward for Non-matching charcater” command
	*	*30*: ASCII code for '0'
	*	*F1*: "Send All Characters" command
	*	*0D*: Carriage return hex value
	*   *Input*: 000037692
	*   *Output*: 37692<CR>

*   **`E7xx`** — **Search Backward for Non-Matching Character**
    *   *Description:* Searches backward for the first character that is **not** `xx`. **Leaves the cursor pointing to the non-`xx` character**.
    *   *Syntax:* `E7xx` (`xx` = character hex to ignore).

---

### Character & String Manipulation

*   **`FBnnxxyy..zz`** — **Suppress Characters**
    *   *Description:* Suppresses all occurrences of up to 15 specified character hex values (`xx`, `yy`, etc.) starting from the current cursor position, as the cursor advances.
    *   *Syntax:* `FBnn[hex_bytes]` (`nn` = decimal count of characters to suppress, 01-15; followed by exact number of 2-digit hex bytes).
	*	*Example*: Remove spaces in barcode data
	*	*FB0120F10D*
	*	*FB*: “Suppress Characters” command
	*	*01*: Number of characters to suppress
	*	*20*: ASCII code for <space>
	*	*F1*: "Send All Characters" command
	*	*0D*: Carriage return hex value
	*   *Input*: 345 678 90
	*   *Output*: 34567890<CR>

*   **`E4nnxx1xx2yy1yy2...`** — **Replace Characters**
    *   *Description:* Replaces up to 15 characters inline without moving the cursor.
    *   *Syntax:* `E4nn[replacements]` (`nn` = total count of search + replacement characters combined (must be even, up to 30); followed by `SearchHex + ReplaceHex` pairs).
	*	*Example*: Replace the zeros in the barcode above with carriage returns.
	*	*E402300DF10D*
	*	*E4*: “Replace Characters” command
	*	*02*: Number of characters, including the target character and the replacement character (0 is replaced by CR, so total characters=2)
	*	*30*: ASCII code for '0'
	*	*0D*: ASCII code for <CR>
	*	*F1*: "Send All Characters" command
	*	*0D*: Carriage return hex value
	*   *Input*: 1234056780ABC
	*   *Output*: 1234<CR>5678<CR>ABC<CR>

*   **`BAnnNN1SS1NN2SS2`** — **Replace String with Another**
    *   *Description:* Searches forward from current position for string `SS1` (length `NN1`) and replaces up to `nn` occurrences with string `SS2` (length `NN2`). **Does not move the cursor**.
    *   *Syntax:* 
        *   `nn`: Count of replacements (`00` = replace all occurrences).
        *   `NN1`: Length of string to be replaced (decimal 01-99).
        *   `SS1`: ASCII hex of string to be replaced.
        *   `NN2`: Length of replacement string (decimal 00-99). **Use `00` to delete the string (SS2 is omitted)**.
        *   `SS2`: ASCII hex of replacement string.
	*	*Example*: Replace “23”s with “ABC”s.
	*	*BA0002323303414243F100*
	*	*BA*: “Replace string with another” command
	*	*00*: count of replacements to be made, 00 means to replace all occurrences of that string
	*	*02*: Length of string to be replaced
	*	*32*: ASCII code for '2'
	*	*33*: ASCII code for '3'
	*	*03*: Length of replacement string 
	*	*41*: ASCII code for 'A'
	*	*42*: ASCII code for 'B'
	*	*43*: ASCII code for 'C'
	*	*F1*: "Send All Characters" command
	*	*00*: ASCII code for <NUL>
	*   *Input*: cd123abc23bc12ab232
	*   *Output*: cd1ABCabcABCbc12abABC2

---

### Keyboards & Interface Modifiers (USB HID Keyboard Only)

*   **`EFnnnn`** — **Insert Delay**
    *   *Description:* Pauses transmission for `nnnn * 5` milliseconds (up to 49,995ms).
    *   *Syntax:* `EFnnnn` (`nnnn` = 4-digit decimal multiplier).

*   **`B5nnssxx[ssxx...]`** — **Insert Keystrokes**
    *   *Description:* Simulates a sequence of physical keystrokes with hardware modifiers.
    *   *Syntax:* `nn` = number of keys pressed without modifiers; followed by pairs of `ss` (Modifier Hex) and `xx` (Key Map Number).
    *   *Key Modifiers (SS) Bit Flags (Can be added together):*
        *   `00` = No Modifier
        *   `01` = Left Shift
        *   `02` = Right Shift
        *   `04` = Left Alt
        *   `08` = Right Alt
        *   `10` = Left Control
        *   `20` = Right Control
        *   *Example modifier addition:* Left Shift (`01`) + Left Alt (`04`) + Left Ctrl (`10`) = `15` hex.

---

## 3. Common ASCII Control Characters Reference

Use these hex values to append delimiters, insert spaces, or simulate action keys:

| Symbol | Hex Value | Name | Common Use |
| :---: | :---: | :---: | :--- |
| **`NUL`** | `00` | Null Character | End-of-string filler, no-op separator |
| **`HT`** | `09` | Horizontal Tab | Delimit data into spreadsheet columns |
| **`LF`** | `0A` | Line Feed | Move down a row |
| **`CR`** | `0D` | Carriage Return | Default Enter key / barcode suffix |
| **`SP`** | `20` | Space | Separate concatenated fields |
| **`0`** | `30` | Digit '0' | Target for removing leading padding zeros |

---

## 4. Translation Examples (Few-Shot Prompting Context)

This section teaches an AI assistant how to write exact command strings using chain-of-logic combinations.

### Example A: Send first 5 characters, insert 1s delay, send the rest
1. **F20500** — Send first 5 characters (`05`), insert Null character (`00`).
2. **EF0200** — Delay `200 * 5ms` = 1000ms = 1 second.
3. **E900** — Send all remaining characters to the end (omit `00` characters at end).
*   **Result Command String:** `F20500EF0200E900`

### Example B: Insert barcode type and length separated by spaces, end with Enter (CR)
1. **B3** — Insert barcode symbology name.
2. **F42001** — Insert Space (`20`), `1` time (`01`).
3. **B4** — Insert barcode length string.
4. **F42001** — Insert Space (`20`), `1` time (`01`).
5. **F10D** — Send all original characters, followed by CR (`0D`).
*   **Result Command String:** `B3F42001B4F42001F10D`

### Example C: Strip all leading zeros, replace all spaces with dashes, send all with CR
1. **E630** — Search forward for the first character that is not Zero (`30`), moving cursor there.
2. **BA000120012D** — Replace string of length `01` containing Space (`20`) with string of length `01` containing Dash (`2D`), count `00` (all occurrences).
3. **F10D** — Send all remaining characters from current cursor, followed by CR (`0D`).
*   **Result Command String:** `E630BA000120012DF10D`

### Example D: Remove first occurrence of string "ERR" and send the rest of data
1. **BA010345525200** — Replace first occurrence (`01`) of 3-character (`03`) string "ERR" (`455252`) with length 00 replacement (delete).
2. **F100** — Send all data, end with NUL (`00`).
*   **Result Command String:** `BA010345525200F100`

### Example E: **ABcde 123$456876%789035bc8**
1. Output of the first 3 characters **F20300**: 
* F2: Send a Number of Characters
* 03: Send 3 char 
* 00: <NUL>. 
* Output: **ABc**

2. Output of the last 3 characters **EAF602F100**: 
* EA: Move cursor to the end
* F6: shift cursor forward 2
* F1: send all 
* 00: <NUL>
* Output: **bc8**

3. Output from position 3 to 7 **F502F20500**
* F5: Move Cursor Forward
* 02: 2 places
* F2: Send a Number of Characters
* 05: 5 charcters
* 00: <NUL>
* Output: **cde 1**

4. Delete 4 characters at the beginning of the string **F504F100**
* F5: Move Cursor Forward
* 04: 4 places
* F1: Send all
* 00: <NUL>
* Output: **e 123$456876%789035bc8**

5. Delete 4 characters at the end of the string **E904**
* E9: Send All But the Last Characters
* 04: 4 characters
* Output: **ABcde 123$456876%78903**

6. Delete characters no. 4 to 7, and 4 characters at the end of the string **F20300F504E904**
* F2: Send a Number of Characters
* 03: Send 3 char 
* 00: <NUL> 
* F5: Move Cursor Forward
* 04: 4 places
* E9: Send All But the Last Characters
* 04: 4 characters
* Output: **ABc23$456876%78903**

7. Insert GH at position 3 **F20200F44701F44801F100**
* F2: Send a Number of Characters
* 02: Send 2 char 
* 00: <NUL>
* F4: Insert a Character Multiple Times
* 47: ASCII for 'G'
* 01: repeat once
* F4: Insert a Character Multiple Times
* 48: ASCII for 'H'
* 01: repeat once
* F1: Send all
* 00: <NUL>
* Output: **ABGHcde 123$456876%789035bc8**

8. After $, insert the string GH **F32400F20100F44701F44801F100**
* F3: Send All Characters Up to a Character
* 24: ASCII for '$'
* 00: <NUL>
* F2: Send a Number of Characters
* 01: 1 char (This is the $)
* 00: <NUL>
* F4: Insert a Character Multiple Times
* 47: ASCII for 'G'
* 01: repeat once
* F4: Insert a Character Multiple Times
* 48: ASCII for 'H'
* 01: repeat once
* F1: Send all
* 00: <NUL>
* Output: **ABcde 123$GH456876%789035bc8**

9. Send everything after $ and delete one % after that **FB0125B0000124F501F100**
* FB: Suppress Characters
* 01: Number of characters to suppress
* 25: ASCII for '%'
* B0: Search Forward for a String
* 0001: Length of string = 1 character
* 24: ASCII for '$'
* F5: Move Cursor Forward
* 01: 1 place
* F1: Send all
* 00: <NUL>
* Output: **456876789035bc8**
