2FA Coin
========
This is a specification for implementing a cryptocurrency that enforces two
factor authentication for any transactions. This would better secure an
individual or enterprise's funds from theft.

## What is Two-Factor Authentication?
Two-factor authentication (also known as 2FA) is a method of confirming a
user's claimed identity by utilizing a combination of two different
components. Two-factor authentication is a type of multi-factor
authentication.

Typically, 2FA is "something you know" and "something you have". A simple
example of this is visiting an ATM. When you go to an ATM you have two things
that are required to pull money out of the machine, your debit card, and your
PIN. The debit card, being "something you have", is a physical object. While
your PIN is "something you know", something that doesn't exist physically 
and only exists in your memory.

Many online websites implement 2FA such as GMail, Facebook, and Coinbase.
While each of these implement their 2FA slightly different from each other,
they generally work the same way and all utilize one time use password. That
is they have the username and password ("something you know") and a
[Time-based One-time Password
Algorithm](https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm)
(also known as TOTP) is "something you have". How this works is by providing a
small randomly generated string (ie "AHV5JG83GQB63M") by the service (ie
GMail) as a "shared secret" between you and the service. You can take that
string along with the current date and time and generate a 6-8 digit pin that
changes every 30 seconds or so (which is what Google Authenticator does).

And here is the problem with this approach of 2FA with a public ledger, you
cannot have a shared secret as you can with a centralized service like GMail.
So using an app like [Google
Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator) will not
work for a decentralized public system (or at least I cannot think of a means
to do so securely).

## Isn't MultiSig 2FA?
No, not really. The reason being all signing keys are "something you have",
since no one memorizes what their keys are. In addition, they are both static and don't support the concept of [HMAC-based
One-time Password
Algorithm](https://en.wikipedia.org/wiki/HMAC-based_One-time_Password_Algorithm) (or HOTP).

## Why Should We Have 2FA for Cryptocurrency Transactions?
Even the most avid crypto enthusiasts will acknowledge that the system is not
perfect. One issue is the ability for the potential for theft. In order to
steal an individual's funds, all that is required is getting access to their
private key (aka wallet). While there are best practices for how to store your
wallet securely, not everyone does it well, and even ones that do still have
that potential of getting their funds stolen.

While its impossible to know the total amount of stolen funds; In the famous
Mt. Gox hack, 650,000 bitcoins were stolen which today (Dec 2017) is worth
over 10 billion dollars. And that is just a single event.  Earlier this month,
NiceHash got another $70 million stolen. In addition to the big guys getting
hacked, individuals have their personal wallets stolen.  One individual
claimed to have $400,000 stolen while being on a public WiFi.

By enabling 2FA on these wallets, having just the private key would not be
enough to steal any of these funds.

In addition, this threat creates FUD (fear, uncertainty, and doubt) in the
community which slows the adoption rate and therefore the price of any
cryptocurrency. In some ways, its not as secure as putting your money in an
institution like a bank (although one could argue that the government can take
your money as Cyprus did in 2013). But the money in the bank of professionally
secured and insured which makes people feel more comfortable about putting
large amounts of money in it.

# How Would 2FA Work?

## Email Enabled Wallets
Each wallet needs a publicly known email address associated with which is
signed with the private key of the wallet. This is stored within the
blockchain (or possibly off-chain on a separate chain). This email address can
be changed, but email addresses declared in the last X blocks are disregarded.
This ensures that a hacker cannot just change the email address to their own
address and transfer funds. It would enforce a delay before the new email
address takes affect (ie 14 days, a month, a year, etc), giving the owner of
the wallet a warning of the email change and move funds to another wallet
before the hacker has an opportunity to steal funds. 

This is also helpful when people change email addresses or if their provider
up and vanishes all of a sudden they don't loose all of their funds.

This email address can be any email address. It can be your personal email
address or you can create a special address just for this wallet purpose. This
would help in maintaining anonymity. One approach could be creating a new
protonmail.com address that is <public key>@protonmail.com (the max length of
an email username is 256 characters).

It is highly recommended that you secure this email with 2FA and a strong
password.

## Two Transactions for One Transfer
In a 2FA transaction blockchain, each transfer requires two transactions 
on the blockchain. Here is what that workflow would look like...

### Workflow

 1) A wallet owner submits a transfer transaction to be committed to the
blockchain. This contains the amount of funds and destination (public key).
This transaction does require a new transaction input format (see below).
Committing this transaction by itself would not actually trigger the funds to
transfer, as a separate confirmation transaction in a separate block is also
required. Adding these transactions to the blockchain does NOT generate funds
to a miner (see Mining Fee below).

 2) The miner (or validator) before committing the transfer transaction to the
blockchain generates a one time use private/public key along with a
transaction ID to be signed by the wallet owner. The block is then committed
with the transaction ID and the one time use public key. An email is sent to
the wallet owner with the private and transaction ID. This email should use
[hashcash](http://www.hashcash.org/) to protect spamming (the email provider
could, in theory, drop any email that does not show proof of work, hence
protecting from spamming activity).

 3) The wallet owner signs the ID with the one time use private key and with their
own permanent private key and submits it to be a confirmation transaction.
By signing twice, it ensures that the miner (or validator) cannot self confirm
a transfer transaction. Even if the miner (or validator) had the wallet owner
private key, it still be extremely difficult to self sign (see Separated
Transactions and Expiration below).

 4) Once the miner gets the signed ID from the wallet owner, they commit a
confirmation transaction. Transfer complete.

Optionally, you could require more than one confirmation transaction which
would increase security but have draw backs as well.

#### Separated Transactions and Expiration

The transfer transaction and confirmation transaction cannot be in the same
block. In addition, the distance between these blocks must not be more than X
blocks (to expire transfer attempts). This ensures that a hacker cannot steal
a private key, create a transfer transaction and confirmation transaction and
mine them into the same block (odds a hacker can commit consecutive blocks is
really unlikely, aka 51% attack). They just also publicly tipped their hand
informing they tried and failed to steal funds. Since we have the contact
email address, the network can email the wallet owner letting them know about
the attempt, giving the owner the opportunity to transfer their funds into a
new wallet.

### New Transaction Input Format

In a standard cryptocurrency transaction, you have to reference past
transactions (aka inputs) of funds going into the wallet to ensure that the
wallet we are sending from has enough funds in it to send the amount of funds.
The new input format now requires only confirmation transactions (which are
linked already to their transfer transactions).

### Mining Fees

Transaction fees only apply to confirmation transactions. This way expired
transactions don't cause the wallet owner anything (ie a hacker cannot steal a
private key and drain their wallet with bogus transfer transactions). This
also incentivize the miners (or validators) to put confirmation transactions
into the block first ensuring less expired transactions. Also, by not putting
transfer transactions into the block, they will not generate the money making
confirmation transactions for themselves later.

# Flaws
 * A decentralized system is now coupled with a centralized system (email)
 * If you loose email access, you temporarily lost access to your funds, until
   you can update your wallet email address.
 * Having multiple transactions to have a single transfer of funds, increases
   the amount time it takes to transfer funds as well as enlarges the
blockchain size.
