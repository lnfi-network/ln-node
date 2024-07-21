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
### For TaprootAssets APIs
✅ `tapd_list_balance`
- ⚠️ coming soon stay tuned

✅ `tapd_send`
- ⚠️ coming soon stay tuned

✅ `tapd_new_addrs`
- ⚠️ coming soon stay tuned

### For P2PMarket APIs
✅ `market_order_book`
- ⚠️ coming soon stay tuned

✅ `market_order_list`
- ⚠️ coming soon stay tuned

✅ `market_order_palce`
- ⚠️ coming soon stay tuned

✅ `market_order_cancel`
- ⚠️ coming soon stay tuned
