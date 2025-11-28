## WhiteBird docs

Documentation for integration with the WhiteBird service

## Contents
- [Integration types](#supported-integration-types)
- [Exchange steps](#required-steps-for-exchange)
- [Merchant concept](#merchant-concept)
- [Identification agent](#user-identification-agent)
- [Limitations](#limitations)
---
### Supported integration types
- **SDK** - out-of-the-box solution allowing the fastest integration of instant exchange functionality into a partner’s service [SDK documentation](./sdk/README.md)
- **API** - backend-to-backend interaction with the WhiteBird service, using the partner’s frontend
- **SDK&API** - hybrid solution, discussed separately.
---
### Required steps for exchange
#### Initial steps:
- User accepts the WhiteBird offer
- Creation of a user account on the WhiteBird platform with up-to-date* personal data
- Waiting for account verification _(may take up to 72 hours, usually much faster)_
- User passes the crypto test _(required only for Belarusian citizens)_
- User adds a payment method to the account _(relevant only for card integrations)_

[API documentation for the above steps](./onboardingAPI/README.md)

#### OnRamp (buying cryptocurrency)
- User selects assets
- User selects payment method
- Adding a wallet for crediting funds (the user must confirm wallet ownership)
- Receiving exchange terms based on selected parameters
- User accepts exchange terms from the above
- Creating an order

During execution, WhiteBird:
- Debits fiat funds from the selected payment method in favor of WhiteBird
- Credits cryptocurrency to the specified wallet
- Completes the order

#### OffRamp (selling cryptocurrency)
- User selects assets
- User selects payout method
- Receiving exchange terms based on selected parameters
- User accepts exchange terms from the above
- Creating an order
- Receiving an address for the user to transfer cryptocurrency  
> maximum time for crediting cryptocurrency is 30 minutes!

During execution, WhiteBird:
- Waits for the cryptocurrency transfer
- Waits for the required number of confirmations
- Credits fiat funds to the selected payment method in favor of the client from WhiteBird
- Completes the order

[OnRamp/OffRamp API documentation](./exchangeAPI/README.md)

---

### Merchant concept

WhiteBird SDK works based on the merchant concept. Users always make transactions on the platform through a merchant. By default, this is WhiteBird itself, but in the case of partner integrations, the partner becomes the merchant. Every WhiteBird client is a merchant for us, which allows flexible configuration:
- personal merchant admin panel
- separate user transaction records
- customizable individual commission terms
- setup of fiat funding/withdrawal methods for clients
- individual merchant revenue terms

---

### User identification agent

Every WhiteBird partner can become a "user identification agent" for us after passing an audit by our AML department.

This enables the partner’s platform to bring its users into WhiteBird without requiring additional registration on WhiteBird by user itself. In this case, the partner performs the user registration.  
WhiteBird receives all necessary registration data via **backend-to-backend** interaction over REST API, using the merchant’s API Key.

---

### Limitations
- All financial transactions must take place directly between WhiteBird and the client. Fiat funds – cards, bank accounts, gaming accounts. Crypto assets – only the client’s crypto wallets.
- From identification agents, **up-to-date** personal data is considered data updated/obtained no more than **three** days before transfer to WhiteBird. _(A simple confirmation by the user at the time of transfer is sufficient to confirm the data is up-to-date.)_
