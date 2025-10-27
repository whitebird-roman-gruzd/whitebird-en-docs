# WhiteBird API Guide

## Process Description

![ExchangeFlow](exchangeApiFlow.drawio.svg)

## Content
- **[Assets](#assets-request)**
- **[Providers](#providers-request)**
- **[Limits](#limit-request)**
- **[Quote](#quote-request)**
- **[Buy](#buy-request---onramp)**
- **[Sell](#sell-request---offramp)**
- **[Order Status](#order-status-request)**

---

### Assets Request

#### POST /api/v2/exchange/merchant/assets
> Currencies available for exchange.

**Parameters:**
- **clientId** - *optional*, string(255), registered client ID

**Response:**
- **cryptoAssets** - [`CryptoAsset`](#common-interfaces), available cryptocurrencies
- **fiatAssets** - [`FiatAsset`](#common-interfaces), available fiat currencies

---

### Providers Request

#### POST /api/v2/exchange/merchant/payment/provider
> List of available payment methods with their configurations.

**Parameters:**
- **clientId** - string(255), registered client ID

**Response:**
- **Array** - [PaymentProvider[]](#paymentprovider-interfaces)

---

### Limit Request

#### POST /api/v2/exchange/merchant/limit
> Client's current limits for the selected currency pair.

**Parameters:**
- **clientId** - string(255)
- **paymentMethod** - PaymentProviderId
- **fromAsset** - LimitAsset
- **toAsset** - LimitAsset

**Response:**
- **asset** - LimitAsset, a copy of `fromAsset`
- **min** - int, minimum amount for exchange
- **max** - int, maximum amount for exchange

**Examples:**
```
// OnRamp example
{
    "clientId": "a03a693e-d6b4-4692-b23d-ccb246392431",
    "paymentMethod": "ASSIST",
    "fromAsset": {
      "code": "BYN"
    },
    "toAsset": {
      "code": "USDT",
      "network": "Ton"
    }
}

// OffRamp example
{
  "clientId": "a03a693e-d6b4-4692-b23d-ccb246392431",
  "paymentMethod": "ASSIST",
  "fromAsset": {
    "code": "USDT",
    "network": "Ton"
  },
  "toAsset": {
    "code": "BYN"
  }
}
```
---

### Quote Request

#### POST /api/v2/exchange/merchant/quote
> Request for deal conditions. Quote is valid for 30 seconds - must be requested continuously. It's not allowed to create an order using an expired quote. The response contains current exchange conditions and a `quoteId`.

**Parameters:**
- `clientId` - *optional*, string(255), registered client ID
- `paymentMethod` - *optional*, PaymentProviderId, which provider to use
- `paymentMethodToken` - *optional*, string(255), card token or any GUID (for custom integrations)
- `fromAsset` - QuoteAsset, asset to exchange from
- `toAsset` - QuoteAsset, asset to exchange to
- `destinationCryptoAddress` - *optional*, string — **OnRamp only**, more accurate network fee estimation
- `comment` - *optional*, string — for **TON OnRamp** transactions

**Response:**
- `quoteId` - string(255), **deal conditions ID**, used in buy/sell requests
- `fromAsset` - QuoteAsset, calculated input amount
- `toAsset` - QuoteAsset, calculated output amount
- `rate` - double, final rate including all fees
- `plainRate` - double, rate without any fees
- `fee` - QuoteFee, fee breakdown
- `expirationDate` - string(255), expiration timestamp (now + 30s)

**Example Requests:**

```
// OnRamp example
{
  "clientId": "a03a693e-d6b4-4692-b23d-ccb246392431",
  "paymentMethod": "TEST",
  "paymentMethodToken": "a13a693e-c6b4-4692-b23d-cdb246392453",
  "fromAsset": {
    "amount": 100,
    "code": "BYN"
  },
  "toAsset": {
    "code": "USDT",
    "network": "Ton"
  },
  "destinationCryptoAddress": "0QD1LvlKFvAT2fHxwm2uCxZ9X5EmIWbGLhGMwDDwcQXauCkU",
  "comment": "12331"
}

// OffRamp example
{
  "clientId": "a03a693e-d6b4-4692-b23d-ccb246392431",
  "paymentMethod": "TEST",
  "paymentMethodToken": "a13a693e-c6b4-4692-b23d-cdb246392453",
  "fromAsset": {
    "amount": 100,
    "code": "USDT",
    "network": "Ton"
  },
  "toAsset": {
    "code": "USD"
  }
}
```
---
### Buy Request – OnRamp

#### POST /api/v2/exchange/merchant/buy
> Request to create an order to buy cryptocurrency using fiat. **OnRamp** operation.

**Parameters:**
- `clientId` - string(255), registered client ID
- `quoteId` - string(255), ID of the deal conditions

**Response:**
- `id` - string(255), order ID
- `type` - `"BUY"`, order type
- `status` - OrderStatus
- `fiatPaymentLink` - string, 3DS link for card integration
- `creationDate` - string(255)
- `modificationDate` - string(255)
- `expiresAtDate` - string(255)

---

### Sell Request – OffRamp

#### POST /api/v2/exchange/merchant/sell
> Request to create an order to sell cryptocurrency into fiat. **OffRamp** operation.

**Parameters:**
- `clientId` - string(255), registered client ID
- `quoteId` - string(255), ID of the deal conditions
- `sourceAddress` - *optional* string(255), crypto address from which the funds will be sent (used to pre-check the wallet)

**Response:**
- `id` - string(255), order ID
- `type` - `"SELL"`, order type
- `status` - OrderStatus
- `depositCryptoAddress` - string, crypto address where client must send the funds
- `creationDate` - string(255)
- `modificationDate` - string(255)
- `expiresAtDate` - string(255)

---

### Order Status Request

#### GET /api/v2/exchange/merchant/order
> Request to get the status of a specific **OnRamp** or **OffRamp** order.

**Parameters:**
- `orderId` - string(255), order ID

**Response:**
- `id` - string(255)
- `number` - int, order number
- `status` - OrderStatus
- `exchangeOperation` - ExchangeOperation, exchange conditions
- `cryptoTransaction` - CryptoTransaction
- `fiatTransaction` - FiatTransaction
- `client` - OrderClient
- `creationDate` - string, UTC
- `modificationDate` - string, UTC
- `expiresAtDate` - string, UTC
- `completionDate` - string, UTC
- `serverDate` - string, UTC (internal info)
- `exchangeType` - `"BUY"` | `"SELL"`
- `operationType` - `"CRYPTO_TO_FIAT"` | `"FIAT_TO_CRYPTO"`
- `orderType` - `"DEFAULT"` (reserved)
- `resultMessage` - *optional* string, service message
- `submitByResident` - boolean, whether the client is a resident of Belarus
- `promoCodeDetails` - string, info on promo code used

---

### Common Interfaces

```typescript
interface FiatAsset {
  id: CurrencyId;
  code: CurrencyCode;
}

interface CryptoAsset {
  id: CurrencyId;
  code: CurrencyCode;
  network: CurrencyNetwork;
  protocol?: CryptoProtocol;  // present for USDT to define the network
}

interface LimitAsset {
  code: CurrencyCode;
  network?: CurrencyNetwork;  // must be provided for crypto
}

interface QuoteAsset {
  // when sending, provide for the asset you are converting from;
  // the response will include both calculated directions
  amount?: double;
  code: CurrencyCode;
  network?: CurrencyNetwork;  // must be provided for crypto
}

interface QuoteFee {
  total: double;           // WhiteBird service fee
  network: double;         // blockchain network fee
  service: null;           // ???
  asset: CurrencyCode;     // fiat currency of the fee
}

enum CurrencyId {
  BYN = "BYN",
  RUB = "RUB",
  USD = "USD",
  EUR = "EUR",
  BTC = "BTC",
  ETH = "ETH",
  USDT_ERC = "USDT_ERC",
  USDC_ERC = "USDC_ERC",
  TON = "TON",
  USDT_TON = "USDT_TON",
  TRX = "TRX",
  USDT_TRC = "USDT_TRC",
}

enum CurrencyCode {
  BYN = "BYN",
  RUB = "RUB",
  USD = "USD",
  EUR = "EUR",
  BTC = "BTC",
  ETH = "ETH",
  USDT = "USDT",
  TON = "TON",
  TRX = "TRX",
  USDC = "USDC",
}

enum CurrencyNetwork {
  Bitcoin = "Bitcoin",
  Ethereum = "Ethereum",
  Ton = "Ton",
  Tron = "Tron",
}

enum CryptoProtocol {
  Erc = "ERC-20",
  Ton = "TON",
  Trc = "TRC-20",
}

enum OrderStatus {
  NEW = "NEW",                // order created
  PROCESSING = "PROCESSING",  // processing in progress
  EXPIRED = "EXPIRED",        // order not fulfilled in time (e.g., crypto not sent by user)
  COMPLETED = "COMPLETED",    // successfully completed
  ERROR = "ERROR"             // execution failed (contact WhiteBird support for details)
}

interface ExchangeOperation {
  inputAsset: number;         // amount of asset being sold
  outputAsset: number;        // amount of asset being received
  exchangeFeeAssetInFiat: number; // service fee amount in fiat
  bonusOutputAsset: number;   // ???
  plainRatio: number;         // rate before fees
  ratio: number;              // rate after fees
  currencyPair: {
    fromCurrency: CurrencyCode;  // sold currency
    toCurrency: CurrencyCode;    // purchased currency
  };
}

interface CryptoTransaction {
  transactionHash?: string;            // blockchain transaction hash
  externalCryptoAddress?: string;      // client wallet address (to/from)
  internalCryptoAddress: string;       // WhiteBird wallet address used in transaction
  fromAddress?: string;                // source address of crypto
  toAddress?: string;                  // destination address of crypto
  status: CryptoTransactionStatus;
  currency: CurrencyCode;              // transaction currency
  comment?: string;                    // for TON network
  fee?: number;                        // blockchain fee
  feeNative?: null;                    // ???
  feePaymentEnabledByClient: boolean;  // whether client pays the network fee
  type: CryptoTransactionType;         // transaction processing type
}

interface FiatTransaction {
  status: FiatTransactionStatus;     // fiat transaction status
  paymentToken: string;              // user's card token/account/unique ID
  internalToken?: string;            // internal service info
  orderIdentity: string;             // internal order ID
  link?: string;                     // 3DS page URL
  providerType: PaymentProviderId;   // payment provider used
  paymentType: string;               // ??? P2P
  processingBank?: string;           // bank used for the transaction (if any)
  resultMessage?: string;            // failure reason
  currency: CurrencyCode;            // transaction currency
  post?: string;                     // ???
  paymentSystem?: PaymentSystemType; // card integration info
}

interface OrderClient {
  clientId: string;
}

enum CryptoTransactionStatus {
  NotFound = "NOT_FOUND"  // no blockchain transaction found
  // ???
}

enum FiatTransactionStatus {
  New = "NEW"   // fiat transaction created
  // ???
}

enum CryptoTransactionType {
  Auto = "AUTO" // processed automatically
  // ???
}
```

### Payment Provider Interfaces

```typescript
interface PaymentProvider {
  id: PaymentProviderId;                     // provider ID
  name: string;
  commissions: PaymentProviderCommission[]; // description of fees
  config: PaymentProviderConfig;            // allowed directions/countries/card types
}

interface PaymentProviderConfig {
  paymentSystems: PaymentSystem[];
}

interface PaymentProviderCommission {
  buyCommission: string;  // percentage
  sellCommision: string;  // percentage
  bank?: string;          // optional, if commission depends on issuing bank
}

interface PaymentSystem {
  paymentSystem: PaymentSystemType;     // card/payment system type
  type: "PSP" | "CI" | "CA";            // PSP = for cards; others = custom integrations
  directions: PaymentDirection[];       // available directions
}

interface PaymentDirection {
  direction: "BUY" | "SELL";
  currencies: PaymentCurrency[];        // allowed currencies + banks + countries
}

interface PaymentCurrency {
  currency: CurrencyCode;
  countries?: string[];
  banks?: string[];
}

enum PaymentProviderId {
  ASSIST = "ASSIST",
  ALFA = "ALFA",
  MTS = "MTS",
  CA = "CA",
  STATUSBANK = "STATUSBANK"
  // or custom values
}

enum PaymentSystemType {
  MIR = "MIR",
  VISA = "VISA",
  BelCard = "BelCard",
  MasterCard = "MasterCard"
}

```
