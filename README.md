# LTO API [![npm version](https://badge.fury.io/js/lto-api.svg)](https://badge.fury.io/js/lto-api)

LegalThings One Platform core features.

## Installation

```js
npm install lto-api --save
```

## Usage

```js
const LTO = require('lto-api').LTO;
const lto = new LTO();
```

### Account

#### Creation
You can create a new random seed with keypair (ed25519):

```js
const account = lto.createAccount();
console.log(account.phrase); // 'satisfy sustain shiver skill betray mother appear pupil coconut weasel firm top puzzle monkey seek'
console.log(account.getSignKeys()); // { privateKey: '4iZ5a5Qx2Utd1432omPUsKXifctCnUr25PjYoR7ohLbnXgG6sazdBg2iXbywzuh6VNWPiFPCudSV2du9HxGxT8mV', publicKey: 'EUkmkWG6TRbsZdQ9UjGySTzkMJq9eaKAjwJpW3Wv6DDH' }

```


#### Recovery 

It's also possible to recover a keypair from an existing seed:

```js
const phrase = 'satisfy sustain shiver skill betray mother appear pupil coconut weasel firm top puzzle monkey seek';

const seed = lto.seedFromExistingPhrase(phrase);
console.log(account.phrase); // 'satisfy sustain shiver skill betray mother appear pupil coconut weasel firm top puzzle monkey seek'
console.log(account.getSignKeys()); // { privateKey: '4iZ5a5Qx2Utd1432omPUsKXifctCnUr25PjYoR7ohLbnXgG6sazdBg2iXbywzuh6VNWPiFPCudSV2du9HxGxT8mV', publicKey: 'EUkmkWG6TRbsZdQ9UjGySTzkMJq9eaKAjwJpW3Wv6DDH' }

```

#### Seed Encryption

Your seed can be encrypted:

```js

const phrase = 'satisfy sustain shiver skill betray mother appear pupil coconut weasel firm top puzzle monkey seek';

const account = lto.seedFromExistingPhrase(phrase);

const password = 'verysecretpassword';
const encrypted = account.encrypt(password); 
console.log(encrypted); //U2FsdGVkX18tLqNbaYdDu5V27VYD4iSylvKnBjMmvQoFFJO1KbsoKKW1eK/y6kqahvv4eak8Uf8tO1w2I9hbcWFUJDysZh1UyaZt6TmXwYfUZq163e9qRhPn4xC8VkxFCymdzYNBAZgyw8ziRhSujujiDZFT3PTmhhkBwIT7FMs=

```

or

```js
const phrase = 'satisfy sustain shiver skill betray mother appear pupil coconut weasel firm top puzzle monkey seek';

const password = 'verysecretpassword';
const encrypted = lto.encryptSeedPhrase(phrase, password);
console.log(encrypted); // U2FsdGVkX18tLqNbaYdDu5V27VYD4iSylvKnBjMmvQoFFJO1KbsoKKW1eK/y6kqahvv4eak8Uf8tO1w2I9hbcWFUJDysZh1UyaZt6TmXwYfUZq163e9qRhPn4xC8VkxFCymdzYNBAZgyw8ziRhSujujiDZFT3PTmhhkBwIT7FMs=
``` 

#### Decryption

To decrypt your seed:

```js
const encryptedSeed = 'U2FsdGVkX18tLqNbaYdDu5V27VYD4iSylvKnBjMmvQoFFJO1KbsoKKW1eK/y6kqahvv4eak8Uf8tO1w2I9hbcWFUJDysZh1UyaZt6TmXwYfUZq163e9qRhPn4xC8VkxFCymdzYNBAZgyw8ziRhSujujiDZFT3PTmhhkBwIT7FMs=';
const password = 'verysecretpassword';
const phrase = lto.decryptSeedPhrase(encryptedSeed);
console.log(phrase); // satisfy sustain shiver skill betray mother appear pupil coconut weasel firm top puzzle monkey seek
```

### EventChain

#### Create an event chain


```js
const chain = account.createEventChain(); // Creates an empty event chain with a valid id and last hash
```

#### Create and sign an event and add it to an existing event chain

```js
const EventChain = require('lto-api').EventChain;
const Event = require('lto-api').Event;

const body = {
  "$schema": "http://specs.livecontracts.io/01-draft/12-comment/schema.json#",
  "identity": {
    "$schema": "http://specs.livecontracts.io/01-draft/02-identity/schema.json#",
    "id": "1bb5a451-d496-42b9-97c3-e57404d2984f"
  },
  "content_media_type": "text/plain",
  "content": "Hello world!"
};

const chain = new EventChain('JEKNVnkbo3jqSHT8tfiAKK4tQTFK7jbx8t18wEEnygya');

chain.addEvent(new Event(body).signWith(account));
```

### HTTPSignature

#### Create a signature for an http request

```js
const headers = {
  date: (new Date("April 1, 2018 12:00:00")).toISOString()
};

const request = new Request('http://example.com', 'get', headers);

const httpSign = new HTTPSignature(request, ['(request-target)', 'date']);
const signatureHeader = httpSign.signWith(account); // keyId="FkU1XyfrCftc4pQKXCrrDyRLSnifX1SMvmx1CYiiyB3Y",algorithm="ed25519-sha256",headers="(request-target) date",signature="tMAxot4iWb8gB4FQ2zqIMfH2Fd8kA9DwSoW3UZPj9f8QlpLX5VvWf314vFnM8MsDo5kqtGzk7XOOy0TL4zVWAg=="
```

## Public Node API

### Transactions

#### Broadcast

Creating a transaction is done with the `Transaction.broadcast` API. There are serveral transaction types you can create:

##### 1. Transfer

To transfer LTO from one account to another.

```js
const transaction = {
  amount: 100000000,
  fee: 10000,
  recipient: '3NARPnCPG4egZbFUQENZ6VDojQqMCpGEG9i',
  attachment: 'foo'
};
const res = await lto.API.PublicNode.transactions.broadcast('transfer', transaction, account.getSignKeys());
/* 
{
  "type" : 4,
  "id" : "GiYdcx3FnTnBMKDxM8DugidxSXSDusJC9rG6BtP8hnCF",
  "sender" : "3N8jr1ytyyYzCVHd8qcoCU3XTFVs24DmYHs",
  "senderPublicKey" : "CcVaW8tKGTY7NDaZ7V6b2phuDUea1mb4xraTBTWmjoFa",
  "fee" : 100000,
  "timestamp" : 1536147726778,
  "signature" : "2Nn7zuHvgR9Zb1NRDB8a6sZLufsS7irQk54hErmqVpjzvGrKfhcmasfVLJrFugk5Li7JcD9A6JWdUKtVBqjmr96P",
  "version" : 1,
  "recipient" : "3NARPnCPG4egZbFUQENZ6VDojQqMCpGEG9i",
  "amount" : 100000,
  "attachment" : "bQbp",
  "height" : 1337
}
*/

```

##### 2. Lease

```js
const transaction = {
  amount: 100000000,
  fee: 10000,
  recipient: '3NARPnCPG4egZbFUQENZ6VDojQqMCpGEG9i'
};
const res = await lto.API.PublicNode.transactions.broadcast('lease', transaction, account.getSignKeys());
```

##### 3. Cancel Lease

```js
const transaction = {
  txId: 'CAQZUgK8bmhvAqWdTV1PUcaLME5KLCUhWC3wcxZopzVc',
  fee: 10000
};
const res = await lto.API.PublicNode.transactions.broadcast('cancelLeasing', transaction, account.getSignKeys());
```

##### 4. Data
```js
const transaction = {
  fee: 10000,
  data: [
    {
      key: '\u2693',
      type: 'binary',
      value: 'base64:' + base64.encode(textBytes)
    }
  ]
};
const res = await lto.API.PublicNode.transactions.broadcast('data', transaction, account.getSignKeys());
```