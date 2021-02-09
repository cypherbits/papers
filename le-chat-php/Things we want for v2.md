Things we want in the new version:

At least the same features as the actual version + Encryption document + this document.

    Code refactor: not compatible with older versions. v2. database version 100.

    Organize code in folders. And do not expose sqlite database to public download on a default installation with default name.

    Remove use of $_REQUEST.

    Remove use of post session and sending the session with GET and manual session cookies. Use native PHP session cookies.

    Recommend use of Snuffleupagus extension:
    -We should use it for PHP session encryption on the server side.
    -User secrets encryption on COOKIES (see the Encryption document).
    -Prevent execution of writeable PHP files. (So we only allow readonly PHP files to be executed, a hacker cannot change our chat code and log passwords).

    Required use of PHP >= 7.4

    Remove use of global variables. Use static in classes.

    Multi chatrooms: public or private. And hidden property (hide from the index page chat list).

    Tabs for private messages or chat rooms??

        Option to whitelist HOST HTTP Header for our domains. (off by default)

        Option to block non-standard User-Agent (on by default).

        Option to only allow Tor Browser User-Agent (off by default).

From opened issues:

    Option to block PMs from guests. DanWin#44

    Cron bot API. DanWin#42

    Native and easy support for emojis and gif.

    DanWin#46

    DanWin#43

    DanWin#40

    DanWin#36

    Themes.

    More: https://github.com/DanWin/le-chat-php/issues

Future work vX features: ??

    2FA TOTP (for members).

    Log every login attempt.

    Sessions list and be able to close them???


## More:

For this: DanWin#53.

Guest sessions private messages will be hidden if the user click "logout" on every session.

If password is not required (is this possible???) we will generate one and show it to the guest user to use it for login on other devices. If the guest user log out on all devices, private messages are hidden because the key is not known.


