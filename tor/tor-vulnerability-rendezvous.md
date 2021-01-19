#Tor core protocol design vulnerabilities and some fixes

## 1. Rendezvous protocol
Applied when connecting to Hidden/Onion Services.

### How it is supposed to work, step by step
1. The client downloads Hidden Service descriptors. We get the Hidden Service Introduction Points from there.
2. Client make a circuit to a random Tor Relay we call the Rendz Relay.
3. Client make a circuit to one of the Introduction Point of the Hidden Service.
4. Client generates a random string called Rendz Cookie.
5. Client sends the Rendz Cookie to the Rendz Relay.
6. Client send INTRODUCE1 cell to the Introduction Point with the Rendz Cookie (encrypted).
7. Introduce Point send INTRODUCE2 cell to the Hidden Service.
8. Hidden Service decrypts the Rendz Cookie and makes a circuit to the Rendz Relay and sends the Rendz Cookie, which matches the string sent by the Client and we can open the communication channel.

### Problems/Vulnerabilities
1. Hidden Service is like a client and does not have full Tor network consensus so it cannot verify if Rendz Relay is valid. Then, we can make it try to connect to any IP Address even if it is not part of the Tor Network.
2. We cannot verify the Client made a circuit to the Rendz Relay and sent the Rendz Cookie. (Point 2 and 5)

## Attackers view
- I can make a 2-hop circuit (instead of 3 for faster speed) to the Introduction Point of the Hidden Service and send Random IP address.
- Close the circuit and start again.
- The Hidden Service will try to connect making the standard 3-hop circuit and fail.
- Attackers need less resources than Hidden Services to attack them.

## Ideas to fix it
While there is already a proposal to implement PoW (https://gitweb.torproject.org/torspec.git/tree/proposals/327-pow-over-intro.txt) and it is a good idea, the base design vulnerability is still there.

###Fix v1:
1. Make Hidden Services have all consensus, so it can verify the Rendz Relay to fix problem 1.
2. Make the Rendz Relay sign the Rendz Cookie so Hidden Service can verify the connection is made. TODO: we need a random/time addition to protect from replay attacks. We can use PoW to generate this cookie too.
3. Having all consensus is a step back for Hidden Services on phones like chat apps. 

###Fix v2:
1. Make Authorities sign each Tor Relay entry. This way Hidden Services does not need the full consensus and just need to verify the signature.

##TODO
- If there is wrong information, or it can be explained better, please, open a ticker.