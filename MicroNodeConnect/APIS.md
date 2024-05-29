MicroNode Connect --> MNConnect
======
`draft` `optional`
## API

### APIs For Taproot Assets Channel
✅ `get_taproot_balance`
✅ `pay_taproot_invoice`
✅ `make_taproot_invoice`

SignMessage
VerifyMessage

pay_instant invoice asset_id amount.

sell amount asset_id to asset_id at price 3.434
buy amount asset_id by asset_id  at price 3242.

### For Lnd APIs
Coming Soon

### For TaprootAssets APIs
Coming Soon

### For P2PMarket APIs

✅ `sell asset`
Request: 
```jsonc
{
    "method": "sell_asset",
    "params": {
        "name":"",
        "owner":"npub1234"
    }
}
```