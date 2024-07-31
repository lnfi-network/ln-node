LN Node Link --> LNLink
======
`draft` `optional`
## API

### APIs For General
✅ `sign_message`
- LND API : https://lightning.engineering/api-docs/api/lnd/lightning/sign-message

Request:
```jsonc
{
    "method": "sign_message",
    "params": {
        "message":"Lnfi Network"
    }
}
```
Response:
```jsonc
{
    "result_type": "sign_message",
    "result": {
        "signature": "Lnfi Network Signed Hex",
    },
}
```
✅ `verify_message`
- ⚠️ coming soon stay tuned
- LND API : https://lightning.engineering/api-docs/api/lnd/lightning/verify-message
- Request:
```jsonc
{
    "method": "verify_message",
    "params": {
        "message":"Lnfi Network",
        "signature":"",
    }
}
```
Response:
```jsonc
{
    "result_type": "verify_message",
    "result": {
        "valid": true,
        "pubkey":"01234"
    },
}

```
### For Taproot Assets APIs(On Lightning Network)
✅ `get_taproot_balance`
- Request:
```jsonc
{
    "method": "get_taproot_balance",
    "params": {
        "asset_id":"123" // the taproot assetId (hex), option
    }
}
```
Response:
```jsonc
{
    "result_type": "get_taproot_balance",
    "result": [
        {
            "asset_id":"123", // the taproot assetId (hex)
            "balance": 1000,  // balance
            "decimal": 2      // decimal (1000 -> 10.00)
        },
        {
            "asset_id":"456",
            "balance": 2000,
            "decimal": 0
        },
    ]
}

```
✅ `make_taproot_invoice`
- Request:
```jsonc
{
    "method": "make_taproot_invoice",
    "params": {
        "asset_id":"0sddsfs", // the taproot assetId (hex)
        "amount": 123, // amount of assets
        "description": "string", // invoice's description, optional
        "description_hash": "string", // invoice's description hash, optional
        "expiry": 213 // expiry in seconds from time invoice is created, optional
    }
}
```
Response:
```jsonc
{
    "result_type": "make_taproot_invoice",
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
        "metadata": {} // generic metadata that can be used to add things like zap/boostagram details for a payer name/comment/etc.
    }
}

```

✅ `pay_taproot_invoice`
- Request:
```jsonc
{
    "method": "pay_taproot_invoice",
    "params": {
        "asset_id":"0sddsfs", // the taproot assetId (hex)
        "invoice": "lnbc50n1...", // bolt11 invoice
        "amount": 123, // invoice amount in msats, optional
    }
}
```
Response:
```jsonc
{
    "result_type": "pay_invoice",
    "result": {
        "preimage": "0123456789abcdef..." // preimage of the payment
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
