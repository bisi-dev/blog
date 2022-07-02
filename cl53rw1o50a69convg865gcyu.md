## Tutorial: Integrate Paystack into your Flutter app

In this article, you are going to learn how to integrate Paystack into your Flutter app.

Paystack offers payment processing software that allows businesses accept payments via credit cards, debit card, money transfer and mobile money. A typical use case is a checkout system for an e-commerce solution in your flutter app.

### **PREREQUISITES**
1. [Android Studio / VSCode + Flutter Plugin](https://docs.flutter.dev/get-started/install)
2. [Paystack Account + API Keys](https://paystack.com/)

### STEP 0: CREATE A FLUTTER PROJECT
If you already have a working app, please skip to next step 👇.
If you don't, start by creating a new flutter project by running the following command in your terminal: 

```bash
# terminal

$ flutter create flutter_paystack_app
``` 

This creates a "hello world" flutter app called "flutter_paystack_app". Open this folder in an editor of your choice.


### STEP 1: ADD YOUR API KEYS TO YOUR PROJECT
It is very unsafe to expose your API keys in a project. It is OK to expose a public key from a cryptographic standpoint, but from a security standpoint, always take great care in protecting all keys to avoid theft and misuse. Therefore, create a .env file in your project's root folder and add your paystack public key to it. 

```env
# .\.env

PUBLIC_KEY=YOUR-PAYSTACK-PUBLIC-KEY
``` 

Include the .env file in your pubspec.yaml to ensure that Flutter recognizes it. You can do so by adding it in the assets list like this:

```yaml
# .\pubspec.yaml

assets:
  - .env
``` 

### STEP 2: INSTALL THE NECESSARY PACKAGES
Add the [flutter_dotenv](https://pub.dev/packages/flutter_dotenv) and [Paystack](https://pub.dev/packages/flutter_paystack) pub package to your pubspec.yaml file. You can do so by including it in the dependencies block like this:

```yaml
# .\pubspec.yaml

dependencies:
  flutter_dotenv: ^5.0.2
  flutter_paystack: ^1.0.5
``` 

Then run the `flutter pub get` command in your project's folder. This would install the flutter_dotenv and Paystack SDK plugin package in your project.


### STEP 3: CREATE YOUR CHECKOUT PAGE AND CARD

Create a simple checkout page with a "Pay with paystack" card. This will act as an interface for a user checkout action. A basic action card in flutter looks like this:

```dart
# .\lib\screens\checkout.dart

Widget CheckOutCard() {
    return InkWell(
      onTap: () {},
      child: Card(),
    );
  }
```

You can find the source code for the entire checkout page UI at this particular step [👉 here](https://github.com/bisi-dev/flutter_paystack_app/blob/f25dd755eac705b0dc2bc4a7a568a9e4c6ea0f8a/lib/screens/checkout.dart).

### STEP 4: IMPORT THE NECESSARY PACKAGES
Connect the flutter_dotenv and Paystack packages to your project. You can use them by adding these lines to the top of your checkout page:

```dart
# .\lib\screens\checkout.dart

import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'package:flutter_paystack/flutter_paystack.dart';
```

### STEP 5: INITIALISE FLUTTER_PLAYSTACK PLUGIN
To access the paystack payment modal, load your API keys and use it to create an instance of the paystack plugin. You can utilise this snippet:
```dart
# .\lib\screens\checkout.dart

final payStackClient = PaystackPlugin();

void _startPaystack () async{
  await dotenv.load(fileName: '.env');
  String? publicKey = dotenv.env['PUBLIC_KEY'];
  payStackClient.initialize(publicKey: publicKey!);
}

@override
void initState() {
  super.initState();
  _startPaystack();
}
```
### DEEP DIVE
This snippet starts with a final variable called `payStackClient` that uses the flutter_paystack package to create a  `PaystackPlugin()` object . Then we create a private async function called `_startPaystack()`. The function makes use of the flutter_dotenv package to load the environment variables in your .env file. This enables us to access our API keys at **"dotenv.env['PUBLIC_KEY']"**. We attach our keys to a nullable string called `publicKey` and use it to create an instance of the Paystack plugin. By calling the _startPaystack function in the `init state` (start) of your checkout page, we can now access the paystack payment modal.

### STEP 6: CREATE PAYSTACK PAYMENT FUNCTION
Create a Paystack payment function in your checkout page.
```dart
# .\lib\screens\checkout.dart

final int amount = 100000;
final String reference = "unique_transaction_ref_${Random().nextInt(1000000)}";

void _makePayment() async {
  final Charge charge = Charge()
    ..email = 'paystackcustomer@qa.team'
    ..amount = amount
    ..reference = reference;

  final CheckoutResponse response = await payStackClient.checkout(context,
      charge: charge, method: CheckoutMethod.card);

  if (response.status && response.reference == reference) {
    ScaffoldMessenger.of(context).showSnackBar(snackBarSuccess);
  } else {
    ScaffoldMessenger.of(context).showSnackBar(snackBarFailure);
  }
}
```
We have reached the crux of this article, please do not be overwhelmed 🙏🙏🙏. We are almost there!!!
### VERY DEEP DIVE
This snippet creates a private async function called `_makePayment()`.

-  With the `charge` variable, we  utilise the Charge class provided by flutter_paystack to prepare a Charge object for the payment modal. We use cascade operators to easily perform a sequence of operations on the same object. This includes adding the `email` (email of the customer to be charged) `amount` (amount to charge in base currency **kobo**) and `reference`(unique transaction reference).

P.S: The reference **MUST** be unique. The paystack instance will not load the payStackClient payment modal if you pass a reference that has already been paid with. In this tutorial we make use of the `dart:math` library to generate a random number. More stringent options can be exercised. You can also add more operators to the Charge object. Customise based on your preference.

![Charge-examples.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1656755790157/JOtnN91Sm.PNG align="center")

- The `response` variable returns a future of CheckOutResponse after calling the paystack object with a `.checkout()` method. There are three methods to choose from: `CheckoutMethod.card`, `CheckoutMethod.selectable`, `CheckoutMethod.bank`. A successful response looks like this:
```json
{message: Success, card: PaymentCard{_cvc: 081, expiryMonth: 6, expiryYear: 23, _type: VERVE, _last4Digits: 7812 , _number: null}, account: null, reference: unique_transaction_ref_553969, status: true, method: CheckoutMethod.card, verify: true}
```
Then, we use this response to determine if a transaction is successful.
If `response.status` is true and `response.response` tallies with ours, it is, and we can provide value to the customer [e.g. navigate to a new page, call an endpoint, write to a database, etc. ]. This tutorial just displays a Snackbar.

### STEP 7: ADD THE FUNCTION IN THE ONTAP CALLBACK OF YOUR CHECKOUT CARD

```dart
# .\lib\screens\checkout.dart

Widget CheckOutCard() {
    return InkWell(
      onTap: () {
        _makePayment();
      },
      child: Card(),
    );
  }
```
This will ensure the payment function is called once your action card is clicked.
Eureka!!! 

![final-pic-photogrid.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656755382991/4YCyk2TRU.png align="center")
You have just built a payment app and can start **"stacking your paper"** 😂😂😂 !!!

![stack-paper.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1656667741311/LLV2pY7As.gif align="center")

Feel free to download/clone the full source code from [👉here](https://github.com/bisi-dev/flutter_paystack_app).
<br>You can also check out the live demo of this tutorial [👉here](https://flutter-portfolio-bisi.netlify.app/#/)
<br>If this helped or you enjoyed this article, leave a like/share to help someone too. Thank you !!!

### MORE
I have added a list of paystack test cards you can use to test your payment app. These cards will only work if you are using a paystack test account i.e test API keys. Find them [👉here](https://github.com/bisi-dev/flutter_paystack_app/blob/main/cards.txt).
<br>I have also included each step as a commit in this GitHub [👉repo](https://github.com/bisi-dev/flutter_paystack_app/commits/main). 
![step-by-step-paystack.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1656667083291/1rTcxBMCv.PNG align="center")
If you are stuck at any step, copy the commit SHA, do a git clone and reset. Then, continue with the rest of the article.
```git
# terminal

$ git clone https://github.com/bisi-dev/flutter_paystack_app.git
$ cd flutter_paystack_app
$ git reset --hard <commit-SHA>
``` 
You can also leave questions/comments. I will duly respond.
Thank you !!!