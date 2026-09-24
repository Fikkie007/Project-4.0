# Indonesian payment provider comparison

Research date: 2026-09-21  
Scope: Future-phase research for an Indonesian marketplace that may later require
virtual accounts, e-wallets, payment webhooks, refunds, seller disbursement, and
marketplace-style fund holding. Fase 1 uses manual fund management without a
payment gateway.

## Recommendation

1. **Shortlist DOKU first if buyer-confirmed release is a hard requirement.** Its official documentation explicitly describes Hold & Release Settlement for marketplaces, including a release API, and separately documents Split Settlement. Both require onboarding/service activation; Split Settlement requires an Aggregator Merchant setup and may require Sales assistance.
2. **Shortlist Xendit first if the priority is marketplace seller onboarding and programmable money movement.** xenPlatform documents sub-accounts, accepting payments for merchants, split routing, balance transfers, and payouts to seller bank accounts. The public docs do not describe a legal escrow or buyer-confirmed hold equivalent, so confirm the permitted holding model with Xendit and counsel.
3. **Keep Duitku and Faspay as operational alternatives.** Both document payment acceptance plus bank/e-wallet disbursement. Their public material does not establish the same marketplace hold/release capability. Duitku explicitly limits disbursement to verified business-entity accounts.
4. **Use Midtrans for a conventional single-merchant checkout, not as the default marketplace funds layer.** It has broad payment methods, webhooks, refunds for selected channels, and merchant withdrawal, but the reviewed public docs do not document seller-level split settlement or escrow-like holding.
5. **Treat iPaymu as a lower-priority discovery candidate.** Its current public API docs cover payment creation and real-time webhooks, but the reviewed official documentation did not expose enough public detail for seller disbursement, refunds, or marketplace holding.

This is a product capability comparison, not a regulatory conclusion. Holding customer funds for later release can affect licensing, contractual, KYC/AML, safeguarding, and consumer-protection obligations. Get written confirmation of the proposed fund flow before implementation.

## Comparison

| Provider | VA + e-wallet acceptance | Webhooks | Refunds | Seller payout / disbursement | Marketplace or escrow-like holding | Approval / commercial caveat | Public pricing found |
|---|---|---|---|---|---|---|---|
| **Xendit** | Yes. Official channel index lists Indonesian bank VAs, GoPay, DANA, OVO, ShopeePay, LinkAja, QRIS, and others. | Yes. Payment, payout, refund, and split-payment webhook documentation is public. | Yes, channel-dependent and returned to the original payment method. Refund webhooks report request success/failure. | Yes. Payout API and xenPlatform seller payouts are documented. | **Platform routing:** sub-accounts, split rules, transfers, and separate balances. **Hold:** no buyer-confirmed escrow/hold feature was identified in the reviewed public docs. | xenPlatform must be activated. Sub-account verification and capability/KYC flows apply. Confirm whether the intended held-balance model is approved. | Public pricing page lists channel fees. Examples visible on 2026-09-21: Indonesian VA IDR 9,000 payment-method fee plus IDR 4,000 Xendit processing fee; GoPay 5% plus IDR 4,000 processing fee; payout fees vary by destination/service. Confirm account-specific pricing. |
| **DOKU** | Yes. Checkout/SNAP docs list multiple bank VAs, DANA, OVO, ShopeePay, i.saku, LinkAja, QRIS, cards, and other channels. | Yes. Public docs cover notification URL setup, notification samples, retries, signatures, and best practices. | Yes for supported flows. Card refunds can be partial and repeated up to the original amount; some card setups require manual processing through Refund Ops or acquiring-bank confirmation. | Yes. KIRIM DOKU documents bank transfer, account inquiry, status, and refund notifications. | **Strongest documented fit:** Hold & Release Settlement explicitly targets marketplaces and releases funds through an API. Split Settlement supports percentage/fixed allocation to registered settlement bank accounts. | Hold/release and split settlement are service features that need activation. Split Settlement requires an Aggregator Merchant model; the docs say to contact Sales if the service is unavailable. | No official public rate card located in the reviewed sources. **Quote-based / confirm with Sales.** |
| **Duitku** | Yes. Payment Gateway docs provide API integrations; official product docs describe Indonesian payment methods including bank transfer/VA and e-wallets. | Payment API details are public; verify the exact callback and retry contract during technical validation. | Refund capability was not sufficiently documented in the reviewed public sources. Confirm channel coverage and whether refunds are API or operational. | Yes. API and batch disbursement to 140+ banks, e-wallets, and Pos Indonesia cash/remittance destinations. | No public marketplace escrow or hold/release feature identified. Disbursement can be used after the marketplace releases funds from its own ledger. | Disbursement is only available to verified business-entity accounts, with a separate API key. Disbursement is IDR-only. | Public pricing was not located in the reviewed sources. **Quote-based / confirm with Sales.** |
| **Faspay** | Yes. Faspay Business advertises 50+ payment methods, including VAs, e-wallets, QRIS, cards, and retail. | Faspay pricing/product pages state real-time VA notification; SendMe provides real-time transfer status and reconciliation. | SendMe supports refund transfers and Paycheck links for refund/disbursement use cases. Confirm whether payment reversal is supported per payment channel. | Yes. SendMe documents API/dashboard transfers to 150+ banks and e-wallets, including mass transfer; prefunding is part of the documented flow. | No public escrow or buyer-confirmed hold/release feature identified. Use platform ledger plus SendMe after release, subject to provider approval. | Enterprise/volume pricing and customized needs are directed to Faspay. Faspay states it is a Bank Indonesia-registered PJP. | Public rate card found. Examples: VA Rp4,000/transaction; QRIS 0.7%; e-wallets 1.5% for OVO/DANA/LinkAja and 2% for ShopeePay; SendMe Rp6,500 to bank and Rp2,000 to e-wallet. Prices exclude VAT where stated. |
| **Midtrans** | Yes. Snap/Core API documents bank transfer, e-wallet, QRIS, cards, retail, and other methods; notification examples include BCA/BNI/BRI/Permata VA, GoPay, ShopeePay, and QRIS. | Yes. HTTP(S) POST notification/webhook URL with signature verification is documented. | Yes for credit card, e-wallet, QRIS, ShopeePay, and Akulaku when settled and payable funds are available; partial/full refund is supported for applicable transactions. Other methods are left to the merchant. | Merchant fund withdrawal/payout is documented. The reviewed public docs do not expose a seller payout API or marketplace sub-account settlement flow. | No public escrow-like hold or seller split settlement identified. Partner/multi-outlet features should not be assumed to provide seller fund custody or release. | Some payment methods and refund access require activation or a support request. Marketplace-specific approval/capability must be confirmed with Midtrans. | Public docs state no implementation/maintenance fee, successful-transaction fees, VAT excluded, and channel-specific fees. Exact rates are linked to a separate official pricing view and may require contact for some channels. |
| **iPaymu** | Payment API docs show a direct payment endpoint and payment methods such as QRIS; account documentation references a merchant VA/API key. The reviewed docs do not provide a complete current VA/e-wallet matrix. | Yes. Official product documentation advertises real-time webhook notifications; API uses signed requests. | No sufficiently detailed public API refund documentation was found in the reviewed sources. A refund policy page exists in the verification navigation; validate the operational/API flow directly with iPaymu. | No sufficiently detailed public disbursement API documentation was found in the reviewed sources. | No public marketplace escrow, split, or hold/release capability identified. | Account verification requires KTP, selfie, bank-account proof, public website, and review estimated at two working days. | No official public rate card found in the reviewed sources. **Quote-based / confirm with Sales.** |

## Capability notes

### DOKU: hold and release

DOKU's official Hold and Release Settlement page gives the closest match to a second-hand marketplace flow:

- Payment request sets `additional_info.hold_settlement: true`.
- Settlement remains held until the platform calls the release endpoint.
- Release can include an `override_settlement` allocation to destination settlement bank accounts.
- DOKU states the funds settle within one business day after release.
- The same page says the feature is suitable for marketplaces and ecommerce businesses.

This should be treated as a provider-controlled settlement hold, not automatically as a legally sufficient escrow arrangement. Confirm the allowed seller/buyer dispute window, maximum hold duration, refund behavior while held, and safeguarding terms in the commercial agreement.

### Xendit: platform routing and seller balances

Xendit's xenPlatform documentation is the clearest fit for a marketplace that needs managed seller accounts:

- Create and manage a sub-account for each seller.
- Accept payments on behalf of sellers.
- Route or split payments to platform and seller accounts.
- Track separate account balances and transaction histories.
- Pay sellers out to bank accounts.

The split-payment documentation also warns that split fees are not automatically returned to the source account when the original payment is refunded. The platform therefore needs explicit refund and commission-reversal rules in its own ledger.

### Provider-neutral implementation requirements

Regardless of provider, the project should keep its own immutable payment ledger and treat provider webhooks as asynchronous evidence, not as the only state store. At minimum, model:

- Provider transaction ID, order ID, seller ID, buyer ID, and idempotency/reference key.
- Payment state separate from settlement state, payout state, refund state, and dispute state.
- Signed webhook verification, replay protection, duplicate handling, and reconciliation polling.
- A release rule for delivered/accepted goods and a separate refund path before and after seller release.
- Fee allocation and recovery rules, especially where a provider does not reverse marketplace commission automatically.

## Sources and access dates

All sources below are first-party provider documentation or official provider product/pricing pages. Accessed 2026-09-21.

### Xendit

- [Available payment channels](https://docs.xendit.co/docs/available-payment-channels.md)
- [Payments API webhooks](https://docs.xendit.co/docs/payments-api-webhooks.md)
- [Refund Payment](https://docs.xendit.co/docs/refund-payment-request.md)
- [Payouts via API](https://docs.xendit.co/docs/payouts-via-api.md)
- [xenPlatform overview](https://docs.xendit.co/docs/xenplatform-overview.md)
- [Accept payments for sub-accounts](https://docs.xendit.co/docs/accepting-payments-for-sub-accounts.md)
- [Route and split payments](https://docs.xendit.co/docs/split-payments.md)
- [Xendit Indonesia pricing](https://www.xendit.co/id/biaya/)

### DOKU

- [DOKU API documentation index](https://developers.doku.com/llms.txt)
- [Supported payment methods](https://developers.doku.com/accept-payments/doku-checkout/configuration/supported-payment-methods.md)
- [Notifications](https://developers.doku.com/get-started-with-doku-api/notification.md)
- [Split Settlement](https://developers.doku.com/accept-payments/finance-and-settlement/split-settlement.md)
- [Hold and Release Settlement](https://developers.doku.com/accept-payments/finance-and-settlement/hold-and-release-settlement.md)
- [Card refund](https://developers.doku.com/accept-payments/direct-api/non-snap/card/refund.md)
- [KIRIM DOKU bank transfer](https://developers.doku.com/payout/kirim-doku/transfer-bank.md)

### Duitku

- [Duitku documentation home](https://docs.duitku.com/)
- [Payment Gateway](https://docs.duitku.com/payment-gateway/overview/)
- [Disbursement](https://docs.duitku.com/disbursement-feature/overview/)
- [Duitku API documentation](https://docs.duitku.com/disbursement/id/)

### Faspay

- [Faspay Business and payment channels](https://faspay.co.id/id/produk/faspay-business/)
- [Faspay pricing](https://faspay.co.id/id/harga/)
- [Faspay SendMe](https://faspay.co.id/id/produk/faspay-sendme/)
- [Faspay SendMe API documentation](https://docs.faspay.co.id/getting-started/faspay-sendme.md)
- [Faspay documentation home](https://docs.faspay.co.id/)

### Midtrans

- [Payment overview](https://docs.midtrans.com/docs/payment-overview.md)
- [HTTP(S) notifications/webhooks](https://docs.midtrans.com/docs/https-notification-webhooks.md)
- [Refund transaction](https://docs.midtrans.com/docs/how-can-i-refund-transaction.md)
- [Receiving funds as payout](https://docs.midtrans.com/docs/receive-your-fund.md)
- [Midtrans payment-service fees](https://docs.midtrans.com/docs/how-much-does-midtrans-charge-for-its-payment-service.md)
- [Midtrans pricing index](https://docs.midtrans.com/docs/pricing.md)

### iPaymu

- [iPaymu API documentation home](https://docs.ipaymu.com/id/docs)
- [iPaymu payment gateway overview](https://docs.ipaymu.com/id)
- [iPaymu account verification](https://docs.ipaymu.com/id/docs/verification)
- [iPaymu verification refund-policy navigation](https://docs.ipaymu.com/id/docs/verification)

## Items to confirm in provider calls

- Whether the provider will approve a second-hand-goods marketplace and its dispute/returns policy.
- Whether funds may be held until buyer acceptance, the maximum hold duration, and what happens on expiry.
- Whether seller onboarding/KYC can be API-driven and whether sellers need separate provider accounts.
- Whether refunds can be initiated before settlement, after settlement, and after a seller payout.
- Payout limits, cut-off times, prefunding requirements, reserve requirements, and reversal behavior.
- Written fee schedule for payment channels, refunds, payouts, VAT, chargebacks, reserves, and minimum monthly commitments.
