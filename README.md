# EmojiObf
This script allows the user to take arbitrary text or data and convert it into emoji, or conversely decode that data from emoji. These can then be used for short message secret writing.

User A:
```
py -3 .\EmojiObf.py -e Help! --type base64
[+] Generating dictionary from emoji base 1f600
[+] Using text from cli: "Help!"
[+] Encoding with base64
[+]    b'SGVscCE'
[+] Encoded data: 😒😆😕😬😜😂😄
```
User A's social media post:
```
Need more coffee 😒😆... Maybe I'm addicted? 😕😬😜😂😄
```
User B:
```
py -3 .\EmojiObf.py -d "Need more coffee 😒😆... Maybe I'm addicted? 😕😬😜😂😄" --type base64
[+] Generating dictionary from emoji base 1f600
[+] Using text from cli: "Need more coffee 😒😆... Maybe I'm addicted? 😕😬😜😂😄"
[+] Decoding with base64
[+] Decoded data: b'Help!'
```
# Help/usage
```
usage: EmojiObf.py [-h] [-e ENCODE] [-d DECODE] [-b BASE] [-s SEED] [-t {custom,hex,base16,base32,base64,base85}] [--alias] [-o OUT]
                   [--mydict MYDICT] [--printdict] [--storedict STOREDICT] [--quiet] [--blacklist BLACKLIST [BLACKLIST ...]]
                   [--customalpha CUSTOMALPHA] [--customwidth CUSTOMWIDTH]

options:
  -h, --help            show this help message and exit
  -e ENCODE, --encode ENCODE
                        Encode mode. Takes either a string or a filepath
  -d DECODE, --decode DECODE
                        Decode mode. Takes either a string or a filepath
  -b BASE, --base BASE  Base unicode number to start looking for emoji at. Smilies are around 1F600, but most code range from 1F300
                        through 1FAF8
  -s SEED, --seed SEED  Use a seed value to shuffle the order the emoji are chosen in. This shuffles the chosen emoji, not the entire
                        emoji database
  -t {custom,hex,base16,base32,base64,base85}, --type {custom,hex,base16,base32,base64,base85}
                        Use a specifc type of encoding. Default is hex (0x00-0xFF).
  --alias               Encode or decode using aliases (which are emoji names with colons)
  -o OUT, --out OUT     Output to a file in utf-32le
  --mydict MYDICT       Rather than generate a dictionary, provide one as a json file
  --printdict           Print the generatae dictionary to screen
  --storedict STOREDICT
                        Store the generated dictionary as a json file
  --quiet               Only print the generated output to screen
  --blacklist BLACKLIST [BLACKLIST ...]
                        List of unicode to avoid (Example "\U0001f6de" or ":wheel:" or "🛞")
  --customalpha CUSTOMALPHA
                        Character string alphabet for "custom" mode. Default is string.ascii_lowercase
  --customwidth CUSTOMWIDTH
                        Emoji bit width for "custom" mode. Default is 9
```
* Encoding and decoding support both CLI input and file input. During decode all non-emoji are ignored, as are emoji outside of the dictionary
* User can adjust the base where the emoji dictionary starts from
* User can provide a seed to randomize the resulting dictionary order (consistently)
* Supporting encodings:
*   Hex - Data is converted to 0x00-0xFF (any input data is supported)
*   base16,base32,base64,base85 - Data encoded using the selected base encoding scheme (any input data is supported)
*   Custom - Does a base-style encoding across emoji icons, using the provided alphabet (which dynamically determines input character width) and "emoji bit width" (which determines how many bits are used within an emoji)
* Emoji alias is supported for both input and output to allow for non-unicode consoles and for clarity
* A custom dictionary can be provided for non-custom encoding types
* A blacklist can be provided of characters to avoid adding into the dictionary (default is currently an empty blacklist)

# Custom encoding
1) A custom dictionary is made based upon the emoji width (respecting the user provided base and seed values). For the default 9 emoji width, this generates an emoji dictionary of 512 in size for the bytes 0b000000000 through 0b111111111. This will be used in the final phase of the encoding.
2) The provided alphabet's size is used to determine the minimum bit width for the alphabet. The default alphabet is string.ascii_lowercase, which is 'abcdefghijklmnopqrstuvwxyz', which is 26 characters plus we add one for the end-of-message, which is all bits set. So, our width for the default alphabet is 5 bits.
3) A dictionary matching the bit values to each character in the alphabet is created for the bit selection. In the above example, this would be 0 through 26, plus eom as 31 (0b00000 through 0b11001, plus eom as 0b11111).
4) A lookup of the input string against this dictionary is done, and the resulting zero-padded bits are all added into a string, with a final eom character at the end. 'hello', using the above example: h=0b00111, e=0b00100, l=0b01011, l=0b01011, o=0b01110, \x00=0b11111; yielding 0b001110010001011010110111011111.
5) This is then padded to the right with '0' for the provided width of the emojis (length of the bits mod 9, in our case): 0b001110010001011010110111011111000000.
6) These bytes are then cut into the emoji width: 0b001110010, 0b001011010, 0b110111011, 0b111000000
7) Finally, the provided encoded bytes are looked up in the custom emoji dictionary and returned to the user per normal.

The decoding process is the reverse of the above process: it looks up the emoji to find the bytes; trims the 0 padding based upon alphabet width; and converts to the look-up characters until the end-of-message value is hit.

# Interest
After PasteSend, I was thinking of various ways that encoding is used and how it's useful for transferring data across a medium where it might otherwise be incompatible. The various Base encodings, MIME, and others were designed to allow complex or binary data to be shuttled across console interfaces and ASCII protocols. I began thinking in another direction: Base64 encoding, for example, encodes three bytes across four bytes, but what if data compatibility wasn't the goal? Emoji have evolved over time, and there are now 3664 emoji as of Unicode 15.0. Emoji are therefore dense but expensive, as we can theoretically encoded across 3664 symbols, but at the cost of 32 bits per character with UTF-32. Although there are more characters available across unicode (149,000+), emoji can innocuously be used without drawing much attention.

# Three Main Ideas
1. Obfuscation - Since the intention is to use short messages, possibly spread over several posts or even several days, it would be very difficult to brute-force attack the messages. The underlying principals are essentially (optionally base encoded) replacement ciphers using emoji rather than characters. Generally, replacement ciphers are difficult to break without the key when only given a few characters of the message. Coupled with the fact that false emoji and information can be placed between the true messages, and it greatly increases the privacy of these messages.
2. In plain sight - Social media is riddled with emoji, and they have infused every age and every culture. It would look abnormal for an English speaker to use Chinese characters or for a Russian speaker to use Korean, but everyone everywhere uses emoji.
3. Adjustability - Even though there is no encryption on the messages, the variation increases the difficulty in identifying the messages, and makes it easy for the users to adjust the obfuscation periodically to futher decrease the chance of their messages being discovered. Changing the type of encoding, randomizing the order emoji are chosen in, and changing the starting offset where emoji are chosen make it so that even knowing that someone is using the tool it would be difficult to decode the text.  

# Future Direction
1. Better automation support - Although the tool works well for one-off encodes and decodes, it would also be good to better support being fed data from web scraping tools and returning decodes to them. Presently this could be done via providing a file input and return a file output, but this could always be done better.
2. Better emoji support - Presently the way EmojiObf interacts with the emoji python library causes some possible emoji to get discarded, reducing the available keyspace. There is also no current support for the expanded emoji that use skin tones, which would be an interesting further way to create data density.
3. Word key system - Presently the communicants need to exchange their settings (similar to any symmetric key system), and this could be cumbersome ("today we're base16 and our seed is 'everest' and our base offset is 1f650"). It would be interesting to develop a system to generate a phrase to communicate the key. This obviously isn't ideal to do via the social media that the emoji are being sent on, but possibly could be used over another medium, like the phone ("How are you doing? I was thinking of having pasta tonight, maybe watch a movie about hiking. Could be fun!").

# Resources
* Video overview: https://youtu.be/80aY-K4CVeo
* General information about emoji and links to useful full list of emojis: https://en.wikipedia.org/wiki/Emoji
* General base64 encoding information used to work on custom encoding: https://en.wikipedia.org/wiki/Base64
* General unicode information: https://en.wikipedia.org/wiki/Unicode
* Emoji library documentation: https://pypi.org/project/emoji/
* Raw Emoji unicode information: https://www.unicode.org/Public/15.0.0/ucd/emoji/emoji-data.txt
