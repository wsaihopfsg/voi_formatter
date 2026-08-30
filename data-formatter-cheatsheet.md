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
    *   *Syntax:* `F1xx` (where `xx` is the trailing insert character's hex value).

*   **`F2nnxx`** — **Send a Number of Characters**
    *   *Description:* Sends `nn` characters starting from the current cursor position, followed by the insert character `xx`. If the end of the data is reached before `nn` characters, it stops sending data and inserts `xx`.
    *   *Syntax:* `F2nnxx` (`nn` = decimal 00-99, `xx` = insert character hex).

*   **`F3ssxx`** — **Send All Characters Up to a Character**
    *   *Description:* Sends all characters starting from the current cursor position up to (but not including) the target character `ss`. Follows with insert character `xx`. **Moves the cursor to `ss`**.
    *   *Syntax:* `F3ssxx` (`ss` = target character hex, `xx` = insert character hex).

*   **`E9nn`** — **Send All But the Last Characters**
    *   *Description:* Sends all characters starting from the current cursor position except for the last `nn` characters of the barcode. **Moves the cursor to 1 position past the last sent character**.
    *   *Syntax:* `E9nn` (`nn` = decimal 00-99).

*   **`F4xxnn`** — **Insert a Character Multiple Times**
    *   *Description:* Inserts character `xx` into the output message `nn` times. **Does not move the cursor**.
    *   *Syntax:* `F4xxnn` (`xx` = character hex, `nn` = decimal repetitions 00-99).

*   **`B3`** — **Insert Symbology Name**
    *   *Description:* Inserts the name of the scanned barcode's symbology (e.g., "Code128") without moving the cursor.
    *   *Syntax:* `B3`

*   **`B4`** — **Insert Barcode Length**
    *   *Description:* Inserts the numeric length of the scanned barcode as a string (omitting leading zeros) without moving the cursor.
    *   *Syntax:* `B4`

---

### Cursor Movement Commands

*   **`F5nn`** — **Move Cursor Forward**
    *   *Description:* Advances the virtual cursor ahead by `nn` characters.
    *   *Syntax:* `F5nn` (`nn` = decimal offset 00-99).

*   **`F6nn`** — **Move Cursor Backward**
    *   *Description:* Rewinds the virtual cursor back by `nn` characters.
    *   *Syntax:* `F6nn` (`nn` = decimal offset 00-99).

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

*   **`F9xx`** — **Search Backward for a Character**
    *   *Description:* Searches backward from the current position for character `xx`. **Leaves the cursor pointing to `xx`**.
    *   *Syntax:* `F9xx` (`xx` = target character hex).

*   **`B0nnnnS`** — **Search Forward for a String**
    *   *Description:* Searches forward from the current position for string `S` of length `nnnn`. **Leaves the cursor pointing to the beginning of string `S`**.
    *   *Syntax:* `B0nnnnS` (`nnnn` = 4-digit decimal length, `S` = ASCII hex bytes of string).

*   **`B1nnnnS`** — **Search Backward for a String**
    *   *Description:* Searches backward from the current position for string `S` of length `nnnn`. **Leaves the cursor pointing to the beginning of string `S`**.
    *   *Syntax:* `B1nnnnS` (`nnnn` = 4-digit decimal length, `S` = ASCII hex bytes of string).

*   **`E6xx`** — **Search Forward for Non-Matching Character**
    *   *Description:* Searches forward for the first character that is **not** `xx`. **Leaves the cursor pointing to the non-`xx` character**.
    *   *Syntax:* `E6xx` (`xx` = character hex to ignore).

*   **`E7xx`** — **Search Backward for Non-Matching Character**
    *   *Description:* Searches backward for the first character that is **not** `xx`. **Leaves the cursor pointing to the non-`xx` character**.
    *   *Syntax:* `E7xx` (`xx` = character hex to ignore).

---

### Character & String Manipulation

*   **`FBnnxxyy..zz`** — **Suppress Characters**
    *   *Description:* Suppresses all occurrences of up to 15 specified character hex values (`xx`, `yy`, etc.) starting from the current cursor position, as the cursor advances.
    *   *Syntax:* `FBnn[hex_bytes]` (`nn` = decimal count of characters to suppress, 01-15; followed by exact number of 2-digit hex bytes).

*   **`E4nnxx1xx2yy1yy2...`** — **Replace Characters**
    *   *Description:* Replaces up to 15 characters inline without moving the cursor.
    *   *Syntax:* `E4nn[replacements]` (`nn` = total count of search + replacement characters combined (must be even, up to 30); followed by `SearchHex + ReplaceHex` pairs).

*   **`BAnnNN1SS1NN2SS2`** — **Replace String with Another**
    *   *Description:* Searches forward from current position for string `SS1` (length `NN1`) and replaces up to `nn` occurrences with string `SS2` (length `NN2`). **Does not move the cursor**.
    *   *Syntax:* 
        *   `nn`: Count of replacements (`00` = replace all occurrences).
        *   `NN1`: Length of string to be replaced (decimal 01-99).
        *   `SS1`: ASCII hex of string to be replaced.
        *   `NN2`: Length of replacement string (decimal 00-99). **Use `00` to delete the string (SS2 is omitted)**.
        *   `SS2`: ASCII hex of replacement string.

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
