##How encryption will work:

On this next generation le-chat-php we will use encryption whenever possible for any private data. We focus on server-side encryption, and this is not safe by itself if the server is compromised for a long time. But we will be describing the best possible method from a server-side view and this should work for protecting non-active members data if the server is compromised and the chat code is changed to log passwords.

All encryption will be done using AES256-GCM and Public/Private crypto using the libsodium library included in PHP.

-----------------------------------------------------------

##[CHAT ROOMS]

The new chat will allow multiple rooms with two types:

- Public: can be encrypted or not. Changing between each mode will delete all old messages. On creating a new encrypted public room: we generate a new random AES-256 key (sodium_crypto_aead_aes256gcm_keygen) and store it on a php generated file (can use base64_encode or sodium_bin2hex to store it). They are always listed on the index page.

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


--------------------------------------------------------------

##[MESSAGES]

Each message on an encrypted chat room will generate a new random IV or Nonce and will be stored alongside the encrypted message on database. Base64 is ok for storing messages and IV.

---------------------------------------------------------------

##[PRIVATE MESSAGES]

Private messages are always encrypted even if the user is guest.




---------------------------------------------------------------

##[USER ENCRYPTION]

Personal notes will be encrypted using personal AES key.

Mods and admin notes will be encrypted using a unique generated AES keys shared with the public keys of the members of that type of users.


--------------------------------------------------------------

##[ENCRYPTED COOKIES]

This means not only cookies with the standard security like:
- Expire time ok.
- Only HTTP enabled.
- HTTPS ?
- SameSite Strict.

But means that the content is encrypted!
The content of the Secure COOKIES sent to the client browser is encrypted with a global AES256 password (set on a chat config file and global for all users). The IV or nonce will be derived from the user PHP session id.

This will be used for storing a private rooms password (second half of the key).

--------------------------------------------------------------


More:
-----

    For this: DanWin#53
    Guest sessions private messages will be hidden if the user click "logout" on every session.
    If password is not required (is this possible???) we will generate one and show it to the guest user to use it for login on other devices. If the guest user log out on all devices, private messages are hidden because the key is not known.



