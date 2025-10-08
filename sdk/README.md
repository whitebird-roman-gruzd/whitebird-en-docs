# SDK Documentation

## Introduction
Welcome to the WhiteBird SDK documentation!

This SDK provides the ability to integrate **WhiteBird instant crypto exchange** functionality directly into your mobile app (Swift/Kotlin SDK) or website (JS SDK).  
Authorization in the SDK uses a simplified mechanism with OTP login to the platform.

The SDK supports interaction with WhiteBird in three modes:
- LoginMode
- AuthMode
- TokensMode

#### Presentation
- You can review the visual flow in [Figma](https://www.figma.com/design/QhTl1W0BEncjvGXRu03UiW/SDK-flow?node-id=0-1&p=f)
- Demo is available at: [SDK Demo](https://sdk.dev.wbdevel.net/v2.0/assets/sdk-demo/index.html)  
  (```MerchantId: 4f19017b-0793-4591-94ff-610bb3c4665b```)

Descriptions of each mode are available [below](#brief-description-of-modes) in this documentation.

## High-level process overview

| General WhiteBird SDK flow | Registration process |
|-|-|
| ![UserFlow](UserFlow.drawio.svg) | ![Registration](Registration.drawio.svg) |

| Verification process | Authorization process                      |
|-|--------------------------------------------|
| ![Verification](verification.drawio.svg) | ![Authorization](Authorization.drawio.svg) |

## Brief description of modes

Terminology:
- Client – the company integrating the SDK
- User – the end user of the product

### LoginMode
This mode is intended for user authentication *(authorization/registration/verification)* via WhiteBird. It allows users to log in with WhiteBird credentials, perform exchange operations, view their transaction history, and contact support. Suitable for scenarios where you need to add the ability to deposit/withdraw funds or exchange cryptocurrency to fiat and back.

### AuthMode
In this mode, the SDK is used for user authentication *(authorization/registration/verification)* via WhiteBird and provides user authorization tokens, which can be used for interaction with the WhiteBird API. In this case, the client is **not** a *user identification agent* for us, and implements their own custom UI for exchange, which makes operations through our API using the client’s authorization tokens.

### TokensMode
This mode is designed to run our SDK with access tokens. It includes all the capabilities of LoginMode but removes WhiteBird authorization.  
It implies a seamless transition from the client’s app to the WhiteBird platform and back. In this mode, the client is assumed to be a *user identification agent* for us.

## User identification agent

The main feature enabled by this is the use of SDK in TokensMode. The user will not need to log into the WhiteBird platform themselves; the client does this on their behalf, obtaining Auth tokens through **backend-to-backend** interaction over REST API, using the merchant’s API Key.

- User registration request: [register](../onboardingAPI/README.md#register-post-request)
- User token request: [generate](../onboardingAPI/README.md#generate-tokens-request)

## Adding to a website

### Connection via CDN
```html
<script src="https://sdk.dev.wbdevel.net/v2.0/integration/wbExchangeSdk-v001.js"></script>
```

### Container
- The SDK content is designed for mobile view – the container should not be less than 360px wide.
- The SDK container can be placed anywhere on the website.
- Pass the HTML element of this container in the configuration.
- !! The SDK does not modify the container’s styles, all container styling is handled by the application.
- The SDK iframe takes up all available space within the container.

### API Reference
Example initialization of SDK in ```LoginMode```:
```javascript
wbExchangeSdk.setup({
  // container for embedding SDK
  el: document.getElementById("wbExchangeSdkWrapper"),
  // client id
  merchantId: 'xxxx',
  mode: wbExchangeSdk.mode.LoginMode,
})
```

Example initialization of SDK in ```TokensMode```:
```javascript
wbExchangeSdk.setup({
  // container for embedding SDK
  el: document.getElementById("wbExchangeSdkWrapper"),
  // client id
  merchantId: 'xxxx',
  mode: wbExchangeSdk.mode.TokensMode,

  accessToken: '*****',
  refreshToken: '*****',
})
```

Example initialization of SDK in ```AuthMode```:
```javascript
wbExchangeSdk.setup({
  // container for embedding SDK
  el: document.getElementById("wbExchangeSdkWrapper"),
  // client id
  merchantId: 'xxxx',
  mode: wbExchangeSdk.mode.AuthMode,

  // callback returns user tokens for interaction with WhiteBird API
  // will not return tokens if the user is not yet verified.
  onLogin: ({accessToken, refreshToken, isUserVerified}) => {},
})
```

### Optional SDK configuration parameters

**externalClientId** - string, should be provided by the merchant to link WhiteBird users with merchant’s users.

**email** - string, allows pre-filling the email field to reduce user actions during WhiteBird login.

**merchantPass** - string, required parameter to prevent session timeout after 15 minutes and allow token refresh.

**currencyAmount** - int, allows pre-filling the currency amount for the exchange.

**currencyFrom** - string from _Currency enum_. Allows pre-filling the currency to be exchanged from.

**currencyTo** - string from _Currency enum_. Allows pre-filling the currency to be exchanged to.
```typescript
let currencyFrom: Currency;
let currencyTo: Currency;

enum Currency {
    BYN,
    RUB,
    USD,
    EUR,
    BTC,
    ETH,
    USDT, // ERC-20
    USDC, // ERC-20
    TRX,
    USDT_TRC, // TRC-20
    TON,
    USDT_TON, // TON
    WBP, // TRC-20
}
```
**cryptoWallet** - string, allows pre-filling the user’s wallet field.

**showBackButtonOnHomePage** - bool, shows a "back" button in our UI and reacts to it using the onExit callback.

**onExit** - () -> void, callback to handle "back" button press.


It is recommended to call ```wbExchangeSdk.cleanup()``` after finishing work with the SDK.
