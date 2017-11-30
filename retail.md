---
title: Retail Payments with Interledger
status: Work In Progress
---

# Retail Payments with Interledger

## Background

Retail payments are a notoriously difficult use case to crack with a new payment method taking on the ubiquity of card payments. Retail payments must be cleared in real-time, and often authorized where one or both parties are offline.

Interledger payments offer the promise of a single universal standard for payments, from any account, to any account, but adoption of ILP for retail payments faces the same challenges.

The promise of "payment channel" based payments is also widely touted as the killer app that will launch systems such as Lightning into the maintsream and thereby making payments via crypto-currency easy and convenient.

## Weaknesses of Card payments

- Debit pull (fraud, cost etc)
- Reach (acquiring infrastructure)

## Weaknesses of Layer 2 systems

- Scale (liquidity locked up in each channel)

## Solution

The solution to scaling ILP for retail payments is to treat sending and receiving payments differently.

The most efficient way to send payments is to use payment channels. If these are used only for sending then it is sufficient to use one-way channels as opposed to bilateral (two-way) channels. With a one-way channel, only the senders liquidity is locked up while the channel is open. 

Wallet holders (customers) establish payment channels with one or more gateways, allocating a portion of their liquidity to each channel. The gateway doesn't need to lock up any liquidity in channels with wallet holders but may establish channels with other gateways or use a payment network that supports ILP natively (like XRP Ledger).

The challenge that must be overcome as a receiver of an ILP payment is being online to fulfill the payment at any time. Fortunately it is straight-forward to delegate this function to another entity without delegating control or providing access to the receiving account.

Wallet holders can delegate this function to any service that is able to observe incoming payments into the receiving account, and produce the fulfillment.

It's possible, but not essential, that the wallet holder uses the same service provider as both their sending gateway and receiving service.

## Protocol

### Enrollment:

The wallet creates a payment channel with one or more gateways and funds this channel. The gateway will expose two standard APIs for both payments and quoting.

The wallet selects one or more receiving accounts and creates a receiving secret which it shares with the receiving service. The receiving service must be able to observe and fulfill incoming prepared transfers on the receiving account.

1. Wallet -> Gateway : Create Channel (Amount, Expiry)
1. Gateway -> Wallet : Create Channel Response (Channel URL base, Sending Secret)
1. Wallet -> Receiving Service : Create Receiver (Account, Receiving Secret)
1. Receiving Service -> Wallet : Create Receiver Response (Receiver URl base)


### Key Rotation

1. Wallet -> Gateway : Change Key (Old Sending Secret)
1. Gateway -> Wallet : Change Key Response (New Sending Secret)
1. Wallet -> Receiving Service : Change Key (New Receiving Secret)
1. Receiving Service -> Wallet : Change Key Response (Ack)


### Payment from Wallet (Online Wallet/Online Payee)

All requests to the gateway are secured by signing them with the Sending Secret. At any time the wallet can close the channel OR rotate the secret.

1. Payee -> Wallet : Payment Requests (ILP Addresses, Amounts, Conditions)
1. Wallet -> Gateway(s) : Get Quote (ILP Address, Amount, Signature)
1. Gateways(s) -> Wallet : Quote Response (Amount, Expiry, Signature)
1. Wallet -> Gateway : Payment (Payment Channel Claim (Amount, Expiry, Signature), ILP Address, Condition)
1. Gateway -> Wallet : Payment Response (Fulfillment, Signature)
1. Wallet -> Payee : Payment Response (ILP Address, Fulfillment)

Note: Getting a quote is optional. The wallet may be confident that it has the data necessary to estimate the costs accurately without the need for a quote.

### Payment from Wallet (Offline Wallet/Online Payee)

All requests to the gateway are secured by signing or encrypting them with the Sending Secret. At any time the wallet can close the channel OR rotate the secret.

1. Payee -> Wallet : Payment Requests (ILP Addresses, Amounts, Conditions)
1. _Wallet generates and encrypts quote requests for each of its gateways_
1. Wallet -> Payee : Encrypted Quote Request(s)
1. Payee -> Gateway(s) : Get Quotes (ILP Address, Amount)
1. Gateways(s) -> Payee : Encrypted Quote Response(s) (Amount, Expiry)
1. Payee -> Wallet : Encrypted Quote Response(s) (Amount, Expiry)
1. Wallet -> Gateway : Payment (Encrypted Payment Channel Claim (Amount, Expiry), ILP Address, Condition)
1. Gateway -> Wallet : Payment Response (Fulfillment)
1. Wallet -> Payee : Payment Response (ILP Address, Fulfillment)


### Payment to Wallet

1. _Wallet generates PaymentRequests. Condition derived from payment data_
1. Wallet -> Payer : PaymentRequest(Single use ILP Address, Amount, Condition)
1. _Payer makes payment via ILP_
1. _Receiving Service is notified of prepared transfer, fulfills it_
1. Payer -> Wallet : Payment Response (Fulfillment, Encrypted Payment Data) 