---
description: Returns a list of addresses owned by client.

{% hint style="warning" %}
Since Alchemy does not store keys, this will always return empty.
{% endhint %}
---
# eth\_accounts

### **Parameters**

none

### **Returns**

`Array of DATA`, 20 Bytes - addresses owned by the client.

#### \*\*\*\*[**Example**](https://composer.alchemyapi.io/?composer_state=%7B%22network%22%3A0%2C%22methodName%22%3A%22eth_accounts%22%2C%22paramValues%22%3A%5B%5D%7D)\*\*\*\*

Request

{% tabs %}
{% tab title="Curl" %}
```bash
curl https://eth-mainnet.alchemyapi.io/v2/your-api-key \
-X POST \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":1}'
```
{% endtab %}

{% tab title="Postman" %}
```http
URL: https://eth-mainnet.alchemyapi.io/v2/your-api-key
RequestType: POST
Body: 
{
    "jsonrpc":"2.0",
    "method":"eth_accounts",
    "params":[],
    "id":1
}
```
{% endtab %}
{% endtabs %}

Result

```javascript
{
  "id":1,
  "jsonrpc": "2.0",
  "result": []
}
```