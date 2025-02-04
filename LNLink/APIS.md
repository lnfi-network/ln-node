LN Node Link --> LNLink
======
`draft` `optional`
## API
https://github.com/nostr-protocol/nips/blob/master/47.md
### Error codes
- `RATE_LIMITED`: The client is sending commands too fast. It should retry in a few seconds.
- `NOT_IMPLEMENTED`: The command is not known or is intentionally not implemented.
- `INSUFFICIENT_BALANCE`: The wallet does not have enough funds to cover a fee reserve or the payment amount.
- `QUOTA_EXCEEDED`: The wallet has exceeded its spending quota.
- `RESTRICTED`: This public key is not allowed to do this operation.
- `UNAUTHORIZED`: This public key has no wallet connected.
- `INTERNAL`: An internal error.
- `OTHER`: Other error.

## Nostr Wallet Connect URI
**client** discovers **wallet service** by scanning a QR code, handling a deeplink or pasting in a URI.

The **wallet service** generates this connection URI with protocol `nostr+walletconnect://` and base path its hex-encoded `pubkey` with the following query string parameters:

- `relay` Required. URL of the relay where the **wallet service** is connected and will be listening for events. May be more than one.
- `secret` Required. 32-byte randomly generated hex encoded string. The **client** MUST use this to sign events and encrypt payloads when communicating with the **wallet service**.
    - Authorization does not require passing keys back and forth.
    - The user can have different keys for different applications. Keys can be revoked and created at will and have arbitrary constraints (eg. budgets).
    - The key is harder to leak since it is not shown to the user and backed up.
    - It improves privacy because the user's main key would not be linked to their payments.
- `lud16` Recommended. A lightning address that clients can use to automatically setup the `lud16` field on the user's profile if they have none configured.

The **client** should then store this connection and use it when the user wants to perform actions like paying an invoice. Due to this NIP using ephemeral events, it is recommended to pick relays that do not close connections on inactivity to not drop events.

### Example connection string
```sh
nostr+walletconnect://b889ff5b1513b641e2a139f661a661364979c5beee91842f8f0ef42ab558e9d4?relay=wss%3A%2F%2Frelay.damus.io&secret=71a8c14c1407c113601079c4302dab36460f0ccd0ad506f1f2dc73b5100e4f3c
```

## Commands

### `pay_invoice`

Description: Requests payment of an invoice.

Request:
```jsonc
{
    "method": "pay_invoice",
    "params": {
        "invoice": "lnbc50n1...", // bolt11 invoice
        "amount": 123, // invoice amount in msats, optional
        "asset_id": "0x1232" // hex 32 bytes string
    }
}
```
<span style="color:green">LNLink Feature : asset_id</span>

Response:
```jsonc
{
    "result_type": "pay_invoice",
    "result": {
        "preimage": "0123456789abcdef...", // preimage of the payment
        "fees_paid": 123, // value in msats, optional
    }
}
```

Errors:
- `PAYMENT_FAILED`: The payment failed. This may be due to a timeout, exhausting all routes, insufficient capacity or similar.

### `multi_pay_invoice`

Description: Requests payment of multiple invoices.

Request:
```jsonc
{
    "method": "multi_pay_invoice",
    "params": {
        "invoices": [
          {"id":"4da52c32a1", "invoice": "lnbc1...", "asset_id": "0x123", "amount": 123}, // bolt11 invoice and amount in msats, amount is optional
          {"id":"3da52c32a1", "invoice": "lnbc50n1...", "asset_id": "0x123 },
        ],
    }
}
```
<span style="color:green">LNLink Feature : asset_id</span>

Response:

For every invoice in the request, a separate response event is sent. To differentiate between the responses, each
response event contains a `d` tag with the id of the invoice it is responding to; if no id was given, then the
payment hash of the invoice should be used.

```jsonc
{
    "result_type": "multi_pay_invoice",
    "result": {
        "preimage": "0123456789abcdef...", // preimage of the payment
        "fees_paid": 123, // value in msats, optional
    }
}
```

Errors:
- `PAYMENT_FAILED`: The payment failed. This may be due to a timeout, exhausting all routes, insufficient capacity or similar.

### `pay_keysend`

Request:
```jsonc
{
    "method": "pay_keysend",
    "params": {
        "amount": 123, // invoice amount in msats, required
        "pubkey": "03...", // payee pubkey, required
        "preimage": "0123456789abcdef...", // preimage of the payment, optional
        "tlv_records": [ // tlv records, optional
            {
                "type": 5482373484, // tlv type
                "value": "0123456789abcdef" // hex encoded tlv value
            }
        ]
    }
}
```

Response:
```jsonc
{
    "result_type": "pay_keysend",
    "result": {
        "preimage": "0123456789abcdef...", // preimage of the payment
        "fees_paid": 123, // value in msats, optional
    }
}
```

Errors:
- `PAYMENT_FAILED`: The payment failed. This may be due to a timeout, exhausting all routes, insufficient capacity or similar.

### `multi_pay_keysend`

Description: Requests multiple keysend payments.

Has an array of keysends, these follow the same semantics as `pay_keysend`, just done in a batch

Request:
```jsonc
{
    "method": "multi_pay_keysend",
    "params": {
        "keysends": [
          {"id": "4c5b24a351", "pubkey": "03...", "amount": 123},
          {"id": "3da52c32a1", "pubkey": "02...", "amount": 567, "preimage": "abc123..", "tlv_records": [{"type": 696969, "value": "77616c5f6872444873305242454d353736"}]},
        ],
    }
}
```

Response:

For every keysend in the request, a separate response event is sent. To differentiate between the responses, each
response event contains a `d` tag with the id of the keysend it is responding to; if no id was given, then the
pubkey should be used.

```jsonc
{
    "result_type": "multi_pay_keysend",
    "result": {
        "preimage": "0123456789abcdef...", // preimage of the payment
        "fees_paid": 123, // value in msats, optional
    }
}
```

Errors:
- `PAYMENT_FAILED`: The payment failed. This may be due to a timeout, exhausting all routes, insufficient capacity or similar.

### `make_invoice`

Request:
```jsonc
{
    "method": "make_invoice",
    "params": {
        "amount": 123, // value in msats
        "description": "string", // invoice's description, optional
        "description_hash": "string", // invoice's description hash, optional
        "expiry": 213, // expiry in seconds from time invoice is created, optional
        "asset_id": "0x123ddddd" // asset_id of taproot assets, 32 bytes hex string
    }
}
```
<span style="color:green">LNLink Feature : asset_id</span>

Response:
```jsonc
{
    "result_type": "make_invoice",
    "result": {
        "type": "incoming", // "incoming" for invoices, "outgoing" for payments
        "invoice": "string", // encoded invoice, optional
        "description": "string", // invoice's description, optional
        "description_hash": "string", // invoice's description hash, optional
        "preimage": "string", // payment's preimage, optional if unpaid
        "payment_hash": "string", // Payment hash for the payment
        "amount": 123, // value in msats
        "fees_paid": 123, // value in msats
        "created_at": unixtimestamp, // invoice/payment creation time
        "expires_at": unixtimestamp, // invoice expiration time, optional if not applicable
        "metadata": {}, // generic metadata that can be used to add things like zap/boostagram details for a payer name/comment/etc.
        "asset_id": "0x1234" // asset_id of taproot assets, 32 bytes hex string
    }
}
```
<span style="color:green">LNLink Feature : asset_id</span>
### `lookup_invoice`

Request:
```jsonc
{
    "method": "lookup_invoice",
    "params": {
        "payment_hash": "31afdf1..", // payment hash of the invoice, one of payment_hash or invoice is required
        "invoice": "lnbc50n1..." // invoice to lookup
    }
}
```

Response:
```jsonc
{
    "result_type": "lookup_invoice",
    "result": {
        "type": "incoming", // "incoming" for invoices, "outgoing" for payments
        "invoice": "string", // encoded invoice, optional
        "description": "string", // invoice's description, optional
        "description_hash": "string", // invoice's description hash, optional
        "preimage": "string", // payment's preimage, optional if unpaid
        "payment_hash": "string", // Payment hash for the payment
        "amount": 123, // value in msats
        "fees_paid": 123, // value in msats
        "created_at": unixtimestamp, // invoice/payment creation time
        "expires_at": unixtimestamp, // invoice expiration time, optional if not applicable
        "settled_at": unixtimestamp, // invoice/payment settlement time, optional if unpaid
        "metadata": {}, // generic metadata that can be used to add things like zap/boostagram details for a payer name/comment/etc.
        "asset_id": "0x1234" // asset_id of taproot assets, 32 bytes hex string
    }
}
```
<span style="color:green">LNLink Feature : asset_id</span>

Errors:
- `NOT_FOUND`: The invoice could not be found by the given parameters.

### `list_transactions`

Lists invoices and payments. If `type` is not specified, both invoices and payments are returned.
The `from` and `until` parameters are timestamps in seconds since epoch. If `from` is not specified, it defaults to 0.
If `until` is not specified, it defaults to the current time. Transactions are returned in descending order of creation
time.

Request:
```jsonc
{
    "method": "list_transactions",
    "params": {
        "from": 1693876973, // starting timestamp in seconds since epoch (inclusive), optional
        "until": 1703225078, // ending timestamp in seconds since epoch (inclusive), optional
        "limit": 10, // maximum number of invoices to return, optional
        "offset": 0, // offset of the first invoice to return, optional
        "unpaid": true, // include unpaid invoices, optional, default false
        "type": "incoming", // "incoming" for invoices, "outgoing" for payments, undefined for both
        "asset_id": "0x1234" // asset_id of taproot assets, 32 bytes hex string
    }
}
```

Response:
```jsonc
{
    "result_type": "list_transactions",
    "result": {
        "transactions": [
            {
               "type": "incoming", // "incoming" for invoices, "outgoing" for payments
               "invoice": "string", // encoded invoice, optional
               "description": "string", // invoice's description, optional
               "description_hash": "string", // invoice's description hash, optional
               "preimage": "string", // payment's preimage, optional if unpaid
               "payment_hash": "string", // Payment hash for the payment
               "asset_id": "", // hex string 32 bytes, Lnlink property
               "amount": 123, // value in msats
               "fees_paid": 123, // value in msats
               "created_at": unixtimestamp, // invoice/payment creation time
               "expires_at": unixtimestamp, // invoice expiration time, optional if not applicable
               "settled_at": unixtimestamp, // invoice/payment settlement time, optional if unpaid
               "metadata": {} // generic metadata that can be used to add things like zap/boostagram details for a payer name/comment/etc.
               "asset_id": "0x1234" // asset_id of taproot assets, 32 bytes hex string
           }
        ],
    },
}
```

### `get_balance`

Request:
```jsonc
{
    "method": "get_balance",
    "params": {}
}
```

Response:
```jsonc
{
    "result_type": "get_balance",
    "result": {
        "balance": 10000, // user's balance in msats
        // change to red
        "assets" : [ // lnlink feature <span style="color:red">"assets"</span>
            {"name": "usdt", "asset_id": "0x12314", "amount": 1234.5, "decimal": 2},
            {"name": "usd", "asset_id": "0x5678", "amount": 567.89, "decimal": 2}
        ]
    }
}
```
<span style="color:green">LNLink Feature : assets</span>

### `get_info`

Request:
```jsonc
{
    "method": "get_info",
    "params": {}
}
```

Response:
```jsonc
{
    "result_type": "get_info",
    "result": {
            "alias": "string",
            "color": "hex string",
            "pubkey": "hex string",
            "network": "string", // mainnet, testnet, signet, or regtest
            "block_height": 1,
            "block_hash": "hex string",
            "methods": ["pay_invoice", "get_balance", "make_invoice", "lookup_invoice", "list_transactions", "get_info", "is_lnlink", "get_assets"], // list of supported methods for this connection
            "notifications": ["payment_received", "payment_sent"], // list of supported notifications for this connection, optional.
    }
}
```
<span style="color:green">lnlink supports node will retrun "is_lnlink"</span>

### `get_assets`

Request:
```jsonc
{
    "method": "get_assets",
    "params": {}
}
```

Response:
```jsonc
{
    "result_type": "get_assets",
    "result": [
        {"asset_id": "0x123", "asset_name":"usdt", "amount": 12345, "decimal":2 },
        {"asset_id": "0x234", "asset_name":"usdc", "amount": 12345, "decimal":2 },
    ]
}
```
<span style="color:green">lnlink supports node will retrun "is_lnlink"</span>
## Notifications

### `payment_received`

Description: A payment was successfully received by the wallet.

Notification:
```jsonc
{
    "notification_type": "payment_received",
    "notification": {
        "type": "incoming",
        "invoice": "string", // encoded invoice
        "description": "string", // invoice's description, optional
        "description_hash": "string", // invoice's description hash, optional
        "preimage": "string", // payment's preimage
        "payment_hash": "string", // Payment hash for the payment
        "amount": 123, // value in msats
        "fees_paid": 123, // value in msats
        "created_at": unixtimestamp, // invoice/payment creation time
        "expires_at": unixtimestamp, // invoice expiration time, optional if not applicable
        "settled_at": unixtimestamp, // invoice/payment settlement time
        "metadata": {}, // generic metadata that can be used to add things like zap/boostagram details for a payer name/comment/etc.
        "asset_id": "0x1234" // asset_id of taproot assets, 32 bytes hex string
    }
}
```

### `payment_sent`

Description: A payment was successfully sent by the wallet.

Notification:
```jsonc
{
    "notification_type": "payment_sent",
    "notification": {
        "type": "outgoing",
        "invoice": "string", // encoded invoice
        "description": "string", // invoice's description, optional
        "description_hash": "string", // invoice's description hash, optional
        "preimage": "string", // payment's preimage
        "payment_hash": "string", // Payment hash for the payment
        "amount": 123, // value in msats
        "fees_paid": 123, // value in msats
        "created_at": unixtimestamp, // invoice/payment creation time
        "expires_at": unixtimestamp, // invoice expiration time, optional if not applicable
        "settled_at": unixtimestamp, // invoice/payment settlement time
        "metadata": {}, // generic metadata that can be used to add things like zap/boostagram details for a payer name/comment/etc.
        "asset_id": "0x1234" // asset_id of taproot assets, 32 bytes hex string
    }
}
```

### For P2PMarket APIs
✅ `market_order_book`
- ⚠️ coming soon stay tuned

✅ `market_order_list`
- ⚠️ coming soon stay tuned

✅ `market_order_palce`
- ⚠️ coming soon stay tuned

✅ `market_order_cancel`
- ⚠️ coming soon stay tuned
