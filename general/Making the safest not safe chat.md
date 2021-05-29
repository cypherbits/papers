# Making the safest not safe chat (WIP)

### by cypherbits

## 1. Changelog

- 2021/02/09: initial version. Work in Progress.

## 2. Description

- Provide a way to implement the safest not safe chat. 
- This means the safest chat that we could develop using only server-side cryptography.
- This idea comes from _le-chat-php_ used on some Dark Webs. Javascript is most of the time disabled, so we cannot use it for real safe end-to-end encryption in the browser. We can think of the safest unsafe way of providing privacy with only server-side crypto.
- We will focus on a general view so any application, not just a chat app can get ideas to get better privacy and security for its users.
- In this case we will be using the standard, and most recommended modern cryptography library: _libsodium_.

## 3. Dictionary

- _IV (Initialization Vector) or Nonce_ is actually the same thing for us. A real random bytes generated for example on each message.

## 4. Individual entities

### 4.1. ENCRYPTED COOKIES

This is one of the key features we will be talking about: they allow us to know a user secret on each request but actually do not store it on our server on plaintext. This is how they work:

1. On user login we generate a new (random, of course) AES-256GCM key with `sodium_crypto_aead_aes256gcm_keygen()` or `random_bytes(SODIUM_CRYPTO_AEAD_AES256GCM_KEYBYTES)`. This will be our SESSION_AES_KEY.
2. SESSION_AES_KEY will be unique only to this new session. It will be stored in the user session storage(for example `$_SESSION`).
3. To generate the IV/Nonce, we can generate it with `random_bytes(SODIUM_CRYPTO_AEAD_AES256GCM_NPUBBYTES)` or derive it from any other known variable, your choice. This will be SESSION_AES_NONCE
4. To create the encrypted data we will be setting in a user cookie sent to the browser we will just use `sodium_crypto_aead_aes256gcm_encrypt()` with our SESSION_AES_KEY and SESSION_AES_NONCE. The result will be serialized to text by using `base64_encode()`.
5. Now the resulting text can be set on a new cookie (using `setcookie()` with safe parameters*).
6. The user browser then MUST make all request with our encrypted cookie.
7. When we want to use this secret data we will `base64_decode()`, then decrypt using `sodium_crypto_aead_aes256gcm_decrypt()`.
8. AES GCM mode comes with tampering protection. If base64_decode or decryption fails we MUST close the user session and delete the data.
9. On server side we can configure PHP to delete session data often. We can even store session data directly on RAM using Memcached. We can use Snuffleupagus PHP extension to even encrypt the content of these user sessions on the server.

***Note:** We suppose in this case you already hardened your cookies with checking the expire time, setting OnlyHTTP, HTTPS setting and the new SameSite=Strict parameter.

### 4.2. CHAT ROOMS

The new chat will allow multiple rooms with two types:

- Public: can be encrypted or not. Changing between each mode will delete all old messages. On creating a new encrypted public room: we generate a new random AES-256 key (`sodium_crypto_aead_aes256gcm_keygen`) and store it on a php generated file (with `base64_encode`). They are always listed on the index page.

- Private: always encrypted. Need a password to join them. Can be listed on the index page or hidden from the public list. Each private chat have a hidden and random ID like "awehf84h4aaa" of at least 8 in length. Admin can add members to these private chats as "invitations". The encryption password is never stored fully on plaintext anymore. Not listed private rooms can be shared with a link containing the chat room ID.

If the chat is listed on the main page:
1. we generate a new random AES-256 key,
2. convert it to text with sodium_bin2hex,
3. grab 32 first characters and store it on plain text on a php file.
4. grab the second part (another 32 characters) and Hash it with SHA512 and store it on the php generated file.
5. The second part of the AES key is used as the room password and is never stored on plaintext on the server.
6. When a member or guest want to join a private chat, they have to enter the password which will be hashed with sha512 and compared with the stored key hash with hash_compare(). If it is correct, we log in the user and generate the full AES key by concatenating the stored on plaintext first half of the AES key with the password validated provided from the user to decrypt all messages. The full AES key is not stored on plaintext anywhere. The user provided password part of the AES key will be stored encrypted in a "ENCRYPTED COOKIES" while the session is on.
7. When the user is a member, we can store the password half of the AES key of the room on database, encrypted with the personal member AES key and encoded in base64 so the member remembers the password. See "USER ENCRYPTION".

Other option for private rooms encryption:
- Do not store the first part of the AES key on plaintext. Use a hash of the full AES key. That means the user have to type or copy/paste a longer key. This may be even safer but may be inconvenient for users.

When we store the room password (second half of the room AES key) on the user SECURE COOKIE, the user does not have tha

### 4.3. MESSAGES

Each message on an encrypted chat room will generate a new random IV or Nonce and will be stored alongside the encrypted message on database. Base64 is ok for storing messages and IV.

### 4.4. PRIVATE MESSAGES

Private messages are always encrypted even if the user is guest.

### 4.5. USER ENCRYPTION

When a user joins: we will generate a USER_KEYPAIR (USER_PUBLICKEY and USER_PRIVATEKEY) and a random USER_AES_KEY.
Members will encrypt their USER_PRIVATEKEY with their USER_AES_KEY before storing it to database.
We can generate a random USER_AES_NONCE too and store it on plaintext.


Personal notes will be encrypted using personal AES key.

Mods and admin notes will be encrypted using a unique generated AES keys shared with the public keys of the members of that type of users.

## 5. Notes
### 5.1. Benchmarking data serialization functions
In benchmarks with PHP 7.4 it shows that:
- `base64_encode()` function is the fastest. (0.00008 seconds)
- `sodium_bin2hex()` function is slower. (0.00019 seconds)
- `sodium_bin2base64`(SODIUM_BASE64_VARIANT_ORIGINAL) function is the slowest. (0.00050 seconds)

### 5.2. Benchmarking key generation
- `random_bytes` seems faster in some cases than `sodium_crypto_aead_aes256gcm_keygen`. But should be the same.