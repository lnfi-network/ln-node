LN Node Link --> LNLink
======
`draft` `optional`

## Rationale
LN Node Link describes a way for clients to access a remote LN Node  through a standardized protocol. Custodians may implement this, or the user may run a bridge that bridges their wallet/node and the Nostr Wallet Connect protocol.

LN Node Link is built on the Nostr Protocol and is also compatible with the NWC protocol ([Nostr Wallet Connect protocol](https://github.com/nostr-protocol/nips/blob/master/47.md)).

## How it works
### Connect To LN Node With LNLink
![image](./images/Tahub.png)

### Work With Lnd(Lightning Support)
![image](./images/Lnd.png)

### Work With Tapd(Taproot Assets Support)
![image](./images/Tapd.png)

## API

### NWC APIs 
LNLink will support all [Nostr Wallet Connect protocol](https://github.com/nostr-protocol/nips/blob/master/47.md) APIs. Currently, it supports the following APIs:

#### Events
✅ `NIP-47 info event`: 13194
✅ `NIP-47 request`: 23194
✅ `NIP-47 response`: 23195

#### APIs (Nostr Wallet Connect)

✅ `get_info`

✅ `get_balance`

✅ `pay_invoice`
- ⚠️ amount not supported (for amountless invoices)
- ⚠️ PAYMENT_FAILED error code not supported

✅ `make_invoice`

✅ `list_transactions`

✅ `lookup_invoice`

#### [Nostr Wallet Connect protocol](./APIS.md)


### General Info APIs

#### initOwner
Request: 
```jsonc
{
    "method": "mn_initOwner",
    "params": {
        "name":"",
        "owner":"npub1234"
    }
}
```

Response:
```jsonc
{
    "result_type": "mn_initOwner",
    "result": {
        "nodeInfo": [
            
        ],
    },
}
```

#### mn_getinfo
Request:
```jsonc
{
    "method": "mn_getinfo",
    "params": {
    }
}
```

Response:
```jsonc
{
    "result_type": "mn_getinfo",
    "result": {
        "nodeInfo": [
            
        ],
    },
}
```

### For Lnd APIs
Coming Soon

### For TaprootAssets APIs
Coming Soon

### For P2PMarket APIs
Coming Soon