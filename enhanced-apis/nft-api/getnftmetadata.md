---
description: >-
  Gets the metadata associated with a given NFT.
---

# getNFTMetadata

{% hint style="info" %}
**NOTE: **This endpoint is currently only available on Mainnet and Testnets for Ethereum and Polygon, as well as [Flow Mainnet](https://docs.alchemy.com/flow/documentation/flow-nft-apis).
{% endhint %}

## Parameters

* `contractAddress`: address of NFT contract
* `tokenId`: Id for NFT (integer)
* `tokenType`: (optional) "ERC721" or "ERC1155", type of token. Note that the request may **perform faster** if this parameter is specified.
* `refreshCache`: (optional) boolean - use `true` to refresh the metadata cache if you think the metadata response is out of date. Noe that adding this param may slow the response down.

## Returns

The getNFTMetadata endpoint returns a JSON Object containing the following fields:

* `contract`: contract for returned NFT
    *  `address`: address of the NFT contract
* `id`: token information
    * `tokenId`: Id for NFT (integer) 
    * `tokenMetadata`
        * `tokenType`: (string) "`ERC721`" or "`ERC1155`"
* `externalDomainViewUrl`: a url representing the location of the NFT's original metadata blob. This is a backup for you to parse when the metadata field is not automatically populated.
* `metadata`: [NOT GUARANTEED TO BE POPULATED] relevant metadata for NFT contract. This is useful for viewing image url, traits, etc. without having to follow the metadata blob url in externalDomainViewUrl to parse manually. 
* `timeLastUpdated`: ISO timestamp of the last cache refresh for the information returned in the metadata field.

{% hint style="danger" %}
**Note on metadata**
Some NFT contracts may not have metadata specified. You may need to parse the `externalDomainViewUrl` response on a case by case basis.  
{% endhint %}

## Example

{% hint style="danger" %}
**Version**
You must update your api_key from your dashboard from V2 --> V1, for example: 
https://eth-mainnet.g.alchemy.com/your-api-key/v2
should be
https://eth-mainnet.g.alchemy.com/your-api-key/v1

**Network**
The examples below is for Ethereum Mainnet. If you are using Polygon you'll need to use your polygon endpoint instead: https://polygon-mainnet.g.alchemy.com/your-api-key/v1/getNFTs...
{% endhint %}

### Request
```
curl 'https://eth-mainnet.g.alchemy.com/your-api-key/v1/getNFTMetadata?contractAddress=0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d&tokenId=2&tokenType=erc721&&refreshCache=true'
```

### Response
```
{
    "contract": {
        "address": "0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d"
    },
    "id": {
        "tokenId": "2",
        "tokenMetadata": {
            "tokenType": "ERC721"
        }
    },
    "externalDomainViewUrl": "ipfs://QmeSjSinHpPnmXmspMjwiXyN6zS4E9zccariGR3jxcaWtq/2",
    "media": {
        "uri": "ipfs://QmcJYkCKK7QPmYWjp4FD2e3Lv5WCGFuHNUByvGKBaytif4"
    },
    "alternateMedia": [{
        "uri": "https://ipfs.io/ipfs/QmcJYkCKK7QPmYWjp4FD2e3Lv5WCGFuHNUByvGKBaytif4"
    }],
    "metadata": {
        "image": "ipfs://QmcJYkCKK7QPmYWjp4FD2e3Lv5WCGFuHNUByvGKBaytif4",
        "attributes": [{
            "trait_type": "Eyes",
            "value": "3d"
        }, {
            "trait_type": "Mouth",
            "value": "Bored Cigarette"
        }, {
            "trait_type": "Fur",
            "value": "Robot"
        }, {
            "trait_type": "Hat",
            "value": "Sea Captain's Hat"
        }, {
            "trait_type": "Background",
            "value": "Aquamarine"
        }]
    },
    "timeLastUpdated": "2022-01-10T20:36:46.666Z"
}
```
