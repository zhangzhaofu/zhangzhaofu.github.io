---
layout: post
title:  "getnewaddress"
date:   2019-10-17 14:58:00 +0800
categories: bitcoin
---

# getnewaddress

## wallet/rpcwallet.cpp
```
static UniValue getnewaddress(const JSONRPCRequest& request)
{
}

static const CRPCCommand commands[] =
{
    // ...

    { "wallet",             "getnewaddress",                    &getnewaddress,                 {"label","address_type"} },
};
```
