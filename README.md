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
Uesr B:
```
py -3 .\EmojiObf.py -d "Need more coffee 😒😆... Maybe I'm addicted? 😕😬😜😂😄" --type base64
[+] Generating dictionary from emoji base 1f600
[+] Using text from cli: "Need more coffee 😒😆... Maybe I'm addicted? 😕😬😜😂😄"
[+] Decoding with base64
[+] Decoded data: b'Help!'
```

# Interest
After PasteSend, I was thinking of various ways that encoding is used and how it's useful for transferring data across a medium where it might otherwise be incompatible. The various Base encodings, MIME, and others were designed to allow complex or binary data to be shuttled across console interfaces and ASCII protocols. I began thinking in another direction: Base64 encoding, for example, encodes three bytes across four bytes, but what if data compatibility wasn't the goal? Emoji have evolved over time, and there are now 3664 emoji as of Unicode 15.0. Emoji are therefore dense but expensive, as we can theoretically encoded across 3664 symbols, but at the cost of 32 bits per character with UTF-32. Although there are more characters available across unicode (149,000+), emoji can innocuously be used without drawing much attention.

# Three Main Ideas
1. Obfuscation - Since the intention is to use short messages, possibly spread over several posts or even several days, it would be very difficult to brute-force attack the messages. The underlying principals are essentially (optionally base encoded) replacement ciphers using emoji rather than characters. Generally, replacement ciphers are difficult to break without the key when only given a few characters of the message. Coupled with the fact that false emoji and information can be placed between the true messages, and it greatly increases the privacy of these messages.
2. In plain sight - Social media is riddled with emoji, and they have infused every age and every culture. It would look abnormal for an English speaker to use Chinese characters or for a Russian speaker to use Korean, but everyone everywhere uses emoji.
3. Adjustability - Even though there is no encryption on the messages, the variation increases the difficulty in identifying the messages, and makes it easy for the users to adjust the obfuscation periodically to futher decrease the chance of their messages being discovered. Changing the type of encoding, randomizing the order emoji are chosen in, and changing the starting offset where emoji are chosen make it so that even knowing that someone is using the tool it would be difficult to decode the text.  

# Future Direction
1. More varied encoding - Presently the encoding is essentially either a straight replacement cipher, or a replacement cipher of the base encoded characters. Converting the input data into binary and using some number of bits of the input to include in single characters would increase the density of the output and reduce the number of required emoji to send a message.
2. Better emoji support - Presently the way EmojiObf interacts with the emoji python library causes some possible emoji to get discarded, reducing the available keyspace. There is also no current support for the expanded emoji that use skin tones, which would be an interesting further way to create data density.
3. Word key system - Presently the communicants need to exchange their settings (similar to any symmetric key system), and this could be cumbersome ("today we're base16 and our seed is 'everest' and our base offset is 1f650"). It would be interesting to develop a system to generate a phrase to communicate the key. This obviously isn't ideal to do via the social media that the emoji are being sent on, but possibly could be used over another medium, like the phone ("How are you doing? I was thinking of having pasta tonight, maybe watch a movie about hiking. Could be fun!").

# Resources
* https://en.wikipedia.org/wiki/Emoji
* https://en.wikipedia.org/wiki/Base64
* https://en.wikipedia.org/wiki/Unicode
* https://pypi.org/project/emoji/
