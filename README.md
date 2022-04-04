# LNURL-withdrawPOS
## A new LNURL-withdraw flow / sub-protocol

**LNURL-withdraw** is a bech32-encoded HTTPS query string or sub-protocol of [LNURL](https://github.com/fiatjaf/lnurl-rfc) which gives the users the ability to receive funds via Lightning Network without the need of manually create an invoice.
<br>Users scan a QRcode or paste the LNURL-withdraw link into their wallet, this does a query to a server and gets a JSON with some info, such as the max amount the user can receive, the min amount the user can request, etc.
Then the wallet tipically asks the user to enter an amount and under the hood it creates a Lightning invoice and sends it to the service provider, which eventually pays that.
<br>So the only things the users have to do are:
1. Scan the QR code / paste the LNURL-withdraw link into the wallet
2. Enter the amount to receive
3. Confirm

**In this repo I propose a new LNURL sub-protocol, I temporary called LNURL-withdrawPOS, which sees the interaction between the user and a POS, possibly equipped with a NFC sensor.**

### Flowchart

![LNURL-withdraw POS (2)_page-0001](https://user-images.githubusercontent.com/38695835/161532078-7266f51c-be0d-4507-8e87-72427b0c1ea8.jpg)

The idea is to simplify the interation between the payer and the payee, for a better UX.
The main actor of the flow here is the merchant/payee, not the payer: the merchant/payee has to simply enter an amount on the POS and this will active the NFC sensor, waiting to receive the LNURL-withdraw link. The payer has to transmit the LNURL-withdrawPOS link, possibly via NFC and doesn't need to confirm the payment on the wallet.

### But we already have LNURL-pay for that!
Yes, we have LNURL-pay and the merchant/payee can already generate a LNURL link / QR code which can be scanned by the payer's wallet. This can then authorize the transaction on its wallet and pay the invoice.
The problem here is that the payer must be online in order to pay the invoice and, as far as I know, cannot delegate the payee for the online part of the flow (GET requests, invoice payment...)

### An "offline" payer.
With this flow the users can make "offline" payments and this completely change the UX of a Lightning payment: they could reserve a part of their funds on a LN wallet for "offline" payments in a LNURL-withdrawPOS string, so they can use it offline, for example when they are abroad and don't have a stable connection or have roaming issues.
They could also define sub-balances in their wallet, so once the reserved funds are ended, no other funds in their wallet is touched.

![LNURL-withdrawPOS in time](https://user-images.githubusercontent.com/38695835/161559784-7ae96a0e-1e61-4ae8-8a0a-f86ee1900d74.png)


#### An example
In t=0, Alice creates a LNURL-withdraw link using her LNURL service (personal server).
She saves the link and moves abroad.

In t=1, Alice needs to pay Bob for his service.
<br>Bob enters the amount on his POS, which is connected to the Internet and stores the amount on a temporary memory, activates the NFC sensor and waits for the LNURL-withdraw link.
<br>Alice sends the link offline, using her phone NFC sensor.
<br>Bob's POS receives the link and starts the negotiation with Alice's server.
<br>If the negotiation is good, Bob receives the payment, otherwise his POS gives an error.
<br>Alice has paid Bob.
