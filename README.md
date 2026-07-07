# CutPriceTiles_OpayoCheckout

Magento 2 module that streamlines checkout for **Cut Price Tiles** so that the
customer is handed straight to the Opayo (Ebizmarts SagePay **Form**) hosted
payment page when they place their order — with **no payment-method selection
step** shown in checkout.

Tested on **Magento 2.4.6-p3** with **Ebizmarts Payment Suite (SagePay/Opayo)**.

## What it does

1. **Hides the payment-method selection UI** at checkout (the method radios and
   the "Payment Method" heading) via a small CSS file loaded only on the
   checkout page. The customer sees just the order summary, billing address,
   terms & conditions, and the Place Order ("Continue to Elavon") button.
2. Because Opayo (SagePay Form) is configured as the **only** active payment
   method, Magento auto-selects it, so Place Order goes directly to Opayo's
   hosted page — using the gateway's standard/default redirect behaviour.
3. On a declined/failed payment the customer stays on the Opayo page and sees
   Opayo's own error message (default gateway behaviour — nothing custom).

The module does **not** modify the Opayo redirect logic itself. It only hides
the selection UI. The "make Opayo the only method" part is a store
configuration change (see below), not code — so it stays fully under admin
control and is easy to reverse.

## Files

```
app/code/CutPriceTiles/OpayoCheckout/
├── registration.php
├── etc/module.xml
└── view/frontend/
    ├── layout/checkout_index_index.xml      # loads the CSS on the checkout page
    └── web/css/opayo-checkout.css            # hides the payment-method selection
```

## Production deployment (Magento production mode)

> Do this in a quiet period. Take a backup / note current settings first.

1. Copy the module into the live codebase:
   `app/code/CutPriceTiles/OpayoCheckout`

2. Enable and upgrade:
   ```
   php bin/magento module:enable CutPriceTiles_OpayoCheckout
   php bin/magento setup:upgrade
   php bin/magento setup:di:compile
   php bin/magento setup:static-content:deploy en_GB -f   # use your store locale(s)
   ```

3. Make Opayo (SagePay Form) the only payment method. **First check which
   methods are currently active on live** (Stores > Configuration > Sales >
   Payment Methods), then disable all except Opayo Form. Example:
   ```
   php bin/magento config:set payment/checkmo/active 0
   php bin/magento config:set payment/paypal_billing_agreement/active 0
   # ...disable any other active methods...
   php bin/magento config:set payment/sagepaysuiteform/active 1
   ```

4. Flush cache and test a real checkout:
   ```
   php bin/magento cache:flush
   ```
   Add a product, go to checkout, confirm there is no payment selection, and
   that Place Order redirects to the live Opayo card page.

## Rollback

```
php bin/magento module:disable CutPriceTiles_OpayoCheckout
# re-enable any payment methods you switched off, e.g.:
php bin/magento config:set payment/checkmo/active 1
php bin/magento cache:flush
```

## Notes

- The CSS intentionally hides the single method's title/radio. If more than one
  payment method is ever re-enabled, un-hiding is just a matter of disabling
  this module (the selection UI returns immediately).
- The billing address and "You will be redirected..." note remain visible, as
  Opayo needs the billing address and it reassures the shopper. These can be
  hidden too on request.
