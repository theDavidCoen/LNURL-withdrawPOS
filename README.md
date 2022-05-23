# LNURL-withdrawPOS
## A LNURL-withdraw flow for POS with NFC sensor (open source implementations)

**LNURL-withdraw** is a bech32-encoded HTTPS query string or sub-protocol of [LNURL](https://github.com/fiatjaf/lnurl-rfc) which gives the users the ability to receive funds via Lightning Network without the need of manually create an invoice.
<br>Users scan a QRcode or paste the LNURL-withdraw link into their wallet, this does a query to a server and gets a JSON with some info, such as the max amount the user can receive, the min amount the user can request, etc.
Then the wallet tipically asks the user to enter an amount and under the hood it creates a Lightning invoice and sends it to the service provider, which eventually pays that.
<br>So the only things the users have to do are:
1. Scan the QR code / paste the LNURL-withdraw link into the wallet
2. Enter the amount to receive
3. Confirm

**In this repo I try to schematize a LNURL-withdraw flow for POS, I temporary called LNURL-withdrawPOS, which sees the interaction between the user and a POS, possibly equipped with a NFC sensor. The idea is to have a common reference for open source implementations.**

### Flowchart

![LNURL-withdraw POS (2)_page-0001](https://user-images.githubusercontent.com/38695835/161532078-7266f51c-be0d-4507-8e87-72427b0c1ea8.jpg)

The idea is to simplify the interation between the payer and the payee, for a better UX.
The main actor of the flow here is the merchant/payee, not the payer: the merchant/payee has to simply enter an amount on the POS and this will active the NFC sensor, waiting to receive the LNURL-withdraw link. The payer has to transmit the LNURL-withdrawPOS link, possibly via NFC and doesn't need to confirm the payment on the wallet.

### But we already have LNURL-pay for that!
Yes, we have LNURL-pay and the merchant/payee can already generate a LNURL link / QR code which can be scanned by the payer's wallet. This can then authorize the transaction on its wallet and pay the invoice.
The problem here is that the payer must be online in order to pay the invoice and, as far as I know, cannot delegate the payee for the online part of the flow (GET requests, invoice payment...)

### An "offline" payer.
With this flow the users can make "offline" payments and this completely change the UX of a Lightning payment: they could reserve a part of their funds on a LN wallet for "offline" payments in a LNURL-withdraw string, so they can use it offline, for example when they are abroad and don't have a stable connection or have roaming issues.
They could also define sub-balances in their wallet, so once the reserved funds are ended, no other funds in their wallet is touched.

![LNURL-withdrawPOS in time](https://user-images.githubusercontent.com/38695835/161559784-7ae96a0e-1e61-4ae8-8a0a-f86ee1900d74.png)


#### An example
In t=0, Alice creates a LNURL-withdraw link using her LNURL service (personal server).
She saves the link and moves abroad.

In t=1, Alice needs to pay Bob for his service.
<br>Bob enters the amount on his POS, which is connected to the Internet and stores the amount on a temporary memory, activates the NFC sensor and waits for the LNURL-withdraw link.
<br>Alice sends the link offline, using her phone NFC sensor or a NFC card (or other physical support).
<br>Bob's POS receives the link and starts the negotiation with Alice's server.
<br>If the negotiation is good, Bob receives the payment, otherwise his POS gives an error.
<br>Alice has paid Bob.

## Differencies between BOLT11 invoice, LNURL-withdraw, LNURL-pay and LNURL-withdrawPOS flows

**Normal BOLT11 invoice:**
- Payer must be online to pay
- Payee/Merchant must be online to receive
- Payee/Merchant doesn't need (but can) to specify an amount to receive. 
- Payee/Merchant tipically shows a QR code or sends a textual Lightning invoice.

**LNURL-pay:**
- Payer must be online to decode the LNURL-pay link and pay
- Payee/Merchant must be online to ask the server to create the LNURL-pay link and receive
- Payee/Merchant doesn't need (but can) to specify an amount to receive. 
- Payee/Merchant tipically shows a QR code or sends a textual LNURL.

**LNURL-withdraw:**
- Payer must be online to ask the server to create the LNURL link
- Payee/Merchant must be online to decode the LNURL-withdraw link and receive
- Payee/Merchant must specify the amount to receive (withdraw) **in the aftermath**, selecting from a range (minWithdrawable/maxWithdrawable)
- Payee/Merchant tipically scans a QR code or requests a textual LNURL.

**LNURL-withdrawPOS:**
- Payer can be offline, its server is always online
- Payee/Merchant must be online to decode the LNURL-withdraw link and receive
- Payee/Merchant must specify the amount to receive (withdraw) **in advance** 
- Payer/Merchant doesn't need to scan the QR code or request a textual LNURL, but just waits for the payer to send the LNURL-withdraw link via NFC

## Security concerns in regards to LNURL-withdraw links
Since a LNURL-withdraw link is static, it's possible for a malicious actor to save the link and use it in t=2 without the owner consensus.
This attack should be prevented and I see two mainly solutions:
1. PIN/Biometric confirmation
2. Disposable cards

Other solutions:<br>
 3. NTAG 424

**PIN/Biometric confirmation**
<br>I think it's possible to add a PIN request to the LNURL-withdraw link, so the user's server asks the POS for a secret (the PIN) and gives an error if no secret/incorrect secret is provided. If the payer uses the app offline, this could require a Biometric confirmation and send the secret to the POS via NFC.
<br>Cons: I believe if the secret is static, it could be saved by the malicious POS and used in t=2. To investigate.

**Disposable links**
<br>Disposable cards are widely adopted by fiat services, such as Revolut, for example, but also by Lightning services with fiat ramp, such as Moon.
A user can create a card and spend the entire amount, then the app forget the card details and it can't be used anymore.
This may also exist here: **whenever the user spend sats using the LNURL-withdraw link, this could be "destroyed" and a new LNURL-withdraw link could be created, which "contains" the change.**

**NTAG 424**
<br>This kind of tags give a great level of interaction and encryption.
<br>This solution would require to implement a checking system server-side, possibly with a new LNbits plugin, so the server can check the LNURL links, add a value to a counter and interact with a private key holded in the card.

## Implementations that use the LNURL-withdrawPOS flow or a similar one
**Please note**: before adding your implementation to this list, be sure it is:
open source, fully interoperable, respects the [LNURL](https://github.com/fiatjaf/lnurl-rfc)-withdraw specs [(LUD03)](https://github.com/fiatjaf/lnurl-rfc/blob/luds/03.md)

 Name | Creator  | Auto-active NFC | Further info |
 ------------ | ------------- | ------------- | ------------- | 
[BTCpay's LNURL NFC Support plugin](https://github.com/btcpayserver/btcpayserver-plugins) | [Andrew Camilleri](https://github.com/Kukks) | NO, merchant needs to tap a button | User taps on "Scan NFC", user taps device with lnurl-withdraw, bolt11 sent to lnurl-w endpoint  |

**I invite the representatives of the implementations to signal their support to this flow by adding themselves to this table.**
## Practical Examples ##
In this video, the payment has been sent to my BTCpay POS, which has a plugin that enables the NFC sensor (LNURL NFC Support plugin by Andrew Camilleri).
<br>Please note that the version is 1.0.1 so it's still early and that I had a bit of difficulty to find the NFC sensor on the back of my phone. The next version of the plugin will have textual feedback to improve the UX.
<br>[![Watch the video](https://img.youtube.com/vi/4m-FQoUAs50/sddefault.jpg)](https://youtu.be/4m-FQoUAs50)
