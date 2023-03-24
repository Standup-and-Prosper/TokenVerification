# External token verification for webhooks
[Standup & Prosper](https://standup-and-prosper.com) use JWTs for authorization to your webhooks, here is an example on how to verify them.

This example applies to **Prosper** which allows for custom webhooks:

```js
const axios = require('axios');
const jwtManager = require('jsonwebtoken');
const jwkConverter = require('jwk-to-pem');

async validateToken(authorizationHeader) {
  let token = authorizationHeader.split(' ')[1];
  let unverifiedToken = jwtManager.decode(token, { complete: true });
  let response = await axios.get('https://api.standup-and-prosper.com/.well-known/jwks');
  let jwk = response.data.keys.find(key => key.kid === unverifiedToken.header.kid);
  let key = jwkConverter(jwk);
  let identity = await jwtManager.verify(token, key, { algorithms: ['RS256'], audience: 'TARGET_SERVICE_URL' });
}
```

The `TARGET_SERVICE_URL` should match the value specified in the webhook.
