# How to update your Ionic Stripe Shop to v3 (Noodlio Pay)

If you have previously downloaded Ionic Stripe Shop, then follow these instructions to update your template to the latest Noodlio Pay API. You can read more about Noodlio Pay [here](https://www.noodl.io/market/product/P201604181926406/noodlio-pay-smooth-payments-with-stripe-accept-payments-without-a-server-side-setup).

## Step 1: Replace the constants in `app.js`

At the top of the file app.js, you'll find the following constants:

```
// remove this
var SERVER_SIDE_URL             = "<SERVER_SIDE_URL>";
var STRIPE_API_PUBLISHABLE_KEY  = "<STRIPE_API_PUBLISHABLE_KEY>";
```

With the new Noodlio Pay API, we don't need this anymore. So remove those lines of code.

## Step 2: Add the new constants

At the place where you removed the constants in **Step 2**, add the following lines of code:

```
// ---------------------------------------------------------------------------------------------------------
// !important settings
// Please fill in the following constants to get the project up and running
// You might need to create an account for some of the constants.

// Obtain your unique Mashape ID from here:
// https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe
var NOODLIO_PAY_API_URL         = "https://noodlio-pay.p.mashape.com";
var NOODLIO_PAY_API_KEY         = "<YOUR-UNIQUE-MASHAPE-ID>";
var NOODLIO_PAY_CHECKOUT_KEY    = {test: "pk_test_QGTo45DJY5kKmsX21RB3Lwvn", live: "pk_live_ZjOCjtf1KBlSHSyjKDDmOGGE"};

// Obtain your unique Stripe Account Id from here:
// https://www.noodl.io/pay/connect
// Please also connect your account on this address
// https://www.noodl.io/pay/connect/test
var STRIPE_ACCOUNT_ID           = "<YOUR-UNIQUE-STRIPE-ID>";

// Define whether you are in development mode (TEST_MODE: true) or production mode (TEST_MODE: false)
var TEST_MODE = true;
```

The `NOODLIO_PAY_API_URL` is basically the location of the server and is fixed. The variable `TEST_MODE` simply takes the values `true` or `false` and defines whether we are in test mode (development) or production (actually charging the user). Now let's define two constants:

**Mashape**

To consume the Stripe Payments API, we'll need to obtain our unique `NOODLIO_PAY_API_KEY`. To do so, head over to [Mashape](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe) and click on the right "Get your API Keys and Start Hacking" or press on "Sign up free".

[<img src="http://noodlio-templates.firebaseapp.com/noodlio-pay/img/mashape-api-keys.png">](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe)

After you are signed in, you'll find your unique API Key in the request example on the [Stripe Payments API page](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe):

```
curl -X POST --include 'https://noodlio-pay.p.mashape.com/charge/token' \
  -H 'X-Mashape-Key: <YOUR-MASHAPE-API-KEY>' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Accept: application/json' \
  ... other values
```

Replace the `NOODLIO_PAY_API_KEY` with this unique identifier.

**Stripe Account**

If you haven't already [sign up for a Stripe Account](https://www.stripe.com). After that, you'll need to retrieve your unique Stripe Account ID (field: `stripe_account`), which you can obtain on the following pages (Note: you'll need to visit both links once):

- For the production mode:
[https://www.noodl.io/pay/connect](https://www.noodl.io/pay/connect)
- For the development mode:
[https://www.noodl.io/pay/connect/test](https://www.noodl.io/pay/connect/test)

The Stripe Account ID looks something like `acct_12abcDEF34GhIJ5K`. Replace the constant `STRIPE_ACCOUNT_ID`.

That's it. Our server is configured and ready to receive payments.

## Step 3: Update the config with the new environments

In the file `app.js`, head over to the part that starts with `.config` and specifically **remove** the following lines of code:

```
// Define your STRIPE_API_PUBLISHABLE_KEY
StripeCheckoutProvider.defaults({key: STRIPE_API_PUBLISHABLE_KEY});
```

and **replace** it with:

```
// Defines your checkout key
switch (TEST_MODE) {
  case true:
    //
    StripeCheckoutProvider.defaults({key: NOODLIO_PAY_CHECKOUT_KEY['test']});
    break
  default:
    //
    StripeCheckoutProvider.defaults({key: NOODLIO_PAY_CHECKOUT_KEY['live']});
    break
};
```


## Step 4: Update the factory `StripeCharge`

Head over to your `services.js` (or wherever you have the factory `StripeCharge`). Add the following line of code at the top of the factory (after the line `var self = this`):

```
// do not add this (just as an illustration)
.factory('StripeCharge', function($q, $http, StripeCheckout) {
  var self = this;

  // add only this
  $http.defaults.headers.common['X-Mashape-Key']  = NOODLIO_PAY_API_KEY;
  $http.defaults.headers.common['Content-Type']   = 'application/x-www-form-urlencoded';
  $http.defaults.headers.common['Accept']         = 'application/json';

  // ...
```

## Step 5: Replace `self.chargeUser()`

Replace `self.chargeUser()` in the factory `StripeCharge` with the following lines of code:

```
self.chargeUser = function(stripeToken, headerData) {
  var qCharge = $q.defer();

  // v3; (previously curlData)
  var param = {
    source: stripeToken,
    amount: Math.floor(headerData.amount*100), // amount in cents
    currency: "usd",
    description: headerData.description,
    stripe_account: STRIPE_ACCOUNT_ID,
    test: TEST_MODE,
  };
  var chargeUrl = NOODLIO_PAY_API_URL + "/charge/token"; //v3
  $http.post(chargeUrl, param)
  .success(
    function(StripeInvoiceData){
      qCharge.resolve(StripeInvoiceData);
      // you can store the StripeInvoiceData for your own administration
    }
  )
  .error(
    function(error){
      console.log(error)
      qCharge.reject(error);
    }
  );
  return qCharge.promise;
};```

# That's it, you're done

You should be able to receive payments now with Stripe. If you are having troubles or issues, please send us an email at `noodlio@seipel-ibisevic.com` with your files and we can always make the changes for you.

Take care, the Noodlio Team
