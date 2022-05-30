# Private Key Retrieval Brain Teaser

## Context

I have generated a public / private key pair using NodeJS's `crypto` library.

I fetched the wallet address from the public key:

```
0x1c1f56e6818bf4d80fe5
```

The algorithm I used to get a wallet address from the public key is:

```typescript
function getWalletAddress(publicKey) {
  const hashedPublicKey = crypto.createHash('sha256').update(publicKey).digest('hex');
  const address = '0x' + hashedPublicKey.substring(0, 20);

  return address;
}
```

I have encrypted a string with the public key and I have turned that encrypted string into base64 format.

The encrypted string in base64 format is:

```
PJC2Vg/dQLOLEn2F+30Fqf47sdtkRtXow7CLHirDSNXdvCailI4m6IGPvDEjaNGEiY2vexJXIfwKtvblkKJhkcIrpqyrDYGHm2FWN2kAo7FFaX3f/ygPtTFO1tafNujHRJDp6ngu0k/xmpkCm9QW0XQYS+q6ILF3Y6thrz/vGlhGWYPrMzO9W2rCMUP/jHEAs5FFNOs/FLSkpkLQH/GtClx8b4agJDsSwErnzMPOH4xHkU+lNCtqLaDTWqdDNCFZDBpmXlPRBgaiptb8oEHDeh+o/Nn5V7+19ZqpQEzXT6q6lSzy/uWwXYUsX4LvAFNH4Wsbkk1uIstCasJOPm8lhg==
```

Then, I have generated a random password of 32 characters.

The available characters for this password are

```
let available = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890!@#$%^&()_';
```

One example password would be

```
^c)#WFz2nk$RqdDBMF#7Foqg9%qN^v8z
```

For each one of these characters, I obtained the ASCII code, like:

```
{
"31": "",      "32": " ",     "33": "!",     "34": "\"",    "35": "#",    
"36": "$",     "37": "%",     "38": "&",     "39": "'",     "40": "(",    
"41": ")",     "42": "*",     "43": "+",     "44": ",",     "45": "-",    
"46": ".",     "47": "/",     "48": "0",     "49": "1",     "50": "2",    
"51": "3",     "52": "4",     "53": "5",     "54": "6",     "55": "7",    
"56": "8",     "57": "9",     "58": ":",     "59": ";",     "60": "<",    
"61": "=",     "62": ">",     "63": "?",     "64": "@",     "65": "A",    
"66": "B",     "67": "C",     "68": "D",     "69": "E",     "70": "F",    
"71": "G",     "72": "H",     "73": "I",     "74": "J",     "75": "K",    
"76": "L",     "77": "M",     "78": "N",     "79": "O",     "80": "P",    
"81": "Q",     "82": "R",     "83": "S",     "84": "T",     "85": "U",    
"86": "V",     "87": "W",     "88": "X",     "89": "Y",     "90": "Z",    
"91": "[",     "92": "\\",    "93": "]",     "94": "^",     "95": "_",    
"96": "`",     "97": "a",     "98": "b",     "99": "c",     "100": "d",    
"101": "e",    "102": "f",    "103": "g",    "104": "h",    "105": "i",    
"106": "j",    "107": "k",    "108": "l",    "109": "m",    "110": "n",    
"111": "o",    "112": "p",    "113": "q",    "114": "r",    "115": "s",    
"116": "t",    "117": "u",    "118": "v",    "119": "w",    "120": "x",    
"121": "y",    "122": "z",    "123": "{",    "124": "|",    "125": "}",    
"126": "~",    "127": ""
}
```

And I have created a folder representing that character under the `contents` folder of this repository.

I have converted the private key to base64, and divided it in exactly 32 chunks. I have encrypted every chunk using the password as symmetric key and init vector:

```typescript
const symmetricKey = Buffer.from(password);
const initVector = Buffer.from(password.substring(0,15));
encryptStringWithSymmetricKey(chunk, symmetricKey, initVector);
```

(Conversely, every chunk can be decrypted using the same method, and if you were to aggregate all of the decrypted chunks you'd get the original base64 string representing the private key).

With this I obtained an array of encrypted chunks of the base64 encoded private key.
Then, I have iterated over the password character by character. I have converted that character to ASCII, and placed the element of the encrypted chunks array at that index in the corresponding folder (inside of the wallet address folder). If a character repeats in the password, a chunk's name increases by 1.

For example:

```
password = 'abcc';
chunks = [1,2,5,7];

a -> 97
b -> 98
c -> 99

97/0x1c1f56e6818bf4d80fe5 would contain "1", the contents of which are chunk 1
98/0x1c1f56e6818bf4d80fe5 would contain "1", the contents of which are chunk 2
99/0x1c1f56e6818bf4d80fe5 would contain "1", the contents of which are chunk 5
99/0x1c1f56e6818bf4d80fe5 would contain "2", the contents of which are chunk 7
```

So now the private key contents are split and located within some subfolders of the `contents` folder. The password is unknown, the private key is unknown.

Assuming you knew the password, the process to retrieve the private key would be (pseudocode):

```
final_string = ''

for every character in password
  folder_name = ascii(character)
  chunk = get_chunk_at_folder(folder_name) --> assume this will return chunks sequentially if n(chunks) > 1
  symmetricKey = Buffer.from(password);
  initVector = Buffer.from(password.substring(0,15));
  final_string += decrypt(chunk, symmetrickey, initvector);

final_string = from_base64_to_utf8(final_string) --> success!!
```

## Problem statement

Using only the information you have at hand (not knowing the password, not knowing anything about the order in which the private key chunks were distributed), can you

* Reconstruct the private key and decrypt the message?
  * What's the message?
* Get the password back?
  * Is knowing the original password the ONLY way to fetch the private key?

Reminder: this is the encrypted message in base64 format:

```
PJC2Vg/dQLOLEn2F+30Fqf47sdtkRtXow7CLHirDSNXdvCailI4m6IGPvDEjaNGEiY2vexJXIfwKtvblkKJhkcIrpqyrDYGHm2FWN2kAo7FFaX3f/ygPtTFO1tafNujHRJDp6ngu0k/xmpkCm9QW0XQYS+q6ILF3Y6thrz/vGlhGWYPrMzO9W2rCMUP/jHEAs5FFNOs/FLSkpkLQH/GtClx8b4agJDsSwErnzMPOH4xHkU+lNCtqLaDTWqdDNCFZDBpmXlPRBgaiptb8oEHDeh+o/Nn5V7+19ZqpQEzXT6q6lSzy/uWwXYUsX4LvAFNH4Wsbkk1uIstCasJOPm8lhg==
```