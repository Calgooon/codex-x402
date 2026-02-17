# x402 Protocol Reference

## BRC-31: Mutual HTTP Authentication

### Handshake
1. Client generates a random nonce (32 bytes, base64-encoded)
2. Client POSTs to `/.well-known/auth` with:
   ```json
   {
     "messageType": "initialRequest",
     "identityKey": "<client-public-key-hex>",
     "nonce": "<client-nonce-base64>"
   }
   ```
3. Server responds with:
   ```json
   {
     "messageType": "initialResponse",
     "identityKey": "<server-public-key-hex>",
     "nonce": "<server-nonce-base64>"
   }
   ```
4. Session is cached locally for 1 hour

### General Messages (Authenticated Requests)
After handshake, every request includes these headers:
- `x-bsv-auth-version`: `"0.1"`
- `x-bsv-auth-identity-key`: client's public key (hex)
- `x-bsv-auth-message-type`: `"generalMessage"`
- `x-bsv-auth-nonce`: client's nonce (base64)
- `x-bsv-auth-your-nonce`: server's nonce (base64)
- `x-bsv-auth-signature`: ECDSA signature (base64)
- `x-bsv-auth-request-id`: unique request ID

### Signature Construction
- Protocol: `[2, "auth message signature"]`
- Key derivation: `keyID = "{client_nonce} {server_nonce}"`
- Payload: binary serialization of HTTP method, URL, headers, body using Bitcoin varints

## BRC-29: HTTP Micropayments

### Flow
1. Client sends authenticated request
2. Server responds 402 with payment headers:
   - `x-bsv-payment-satoshis-required`: amount in satoshis
   - `x-bsv-payment-derivation-prefix`: prefix for key derivation
   - `x-bsv-payment-version`: `"1.0"`
3. Client creates a payment transaction via MetaNet Client wallet:
   - Protocol ID: `[2, "3241645161d8"]`
   - derivationSuffix: base64-encoded random bytes
   - Output at index 0 pays the server
   - `randomizeOutputs: false` (critical — server checks index 0)
4. Client retries the request with `x-bsv-payment` header containing:
   ```json
   {
     "derivationPrefix": "<from-402-header>",
     "derivationSuffix": "<base64-random>",
     "transaction": "<base64-encoded-raw-tx>"
   }
   ```
5. Server verifies payment and returns the result

### Refunds
If a paid request fails after payment, the server may include a refund in the response body:
```json
{
  "refund": {
    "derivationPrefix": "...",
    "derivationSuffix": "...",
    "transaction": "<base64-atomic-beef>"
  }
}
```
The client auto-internalizes refunds via MetaNet Client's `internalizeAction`.

## Agent Registry

URL: `https://x402agency.com/.well-known/agents`

Returns JSON with an `agents` array. Each agent has:
- `name`: short name for resolution (e.g., `banana`)
- `url`: base URL for API requests
- `tagline`: brief description
- `x402_info` (optional): URL to hosted x402-info manifest

The registry is cached locally for 5 minutes.
