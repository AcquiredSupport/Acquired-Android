# Acquired-Android

## Description
The Acquired-Android SDK allows you to easily and securely integrate Google Pay into your Android app.

## Requirements
- Android 
- Android Gradle Plugin
- Gradle 

## Install the SDK
Use gradle to install this SDK.

```gradle
repositories {
    maven {
        url = uri("https://maven.pkg.github.com/AcquiredSupport/Acquired-Android")
        credentials {
            username = USERNAME
            password = TOKEN
        }
    }
}

dependencies {
  implementation 'com.acquired.paymentgateway:paymentgateway:VERSION'
}
```

## How to use the SDK

### Configuration
Create an instance of the PaymentGateway with a configuration:
```kotlin
val configuration = Configuration(
    companyId = "459",
    companyPass = "re3vKdCG",
    companyHash = "cXaFMLbH",
    companyMidId = "1687",
    baseUrlAddress = URL("https://qaapi.acquired.com/"),
    requestRetryAttempts = 3
)

val paymentGateway = PaymentGateway(configuration)
```
### Start the Payment
Get the payment data as configured in the Hub:
```kotlin
viewModelScope.launch {
    paymentGateway.getPayment().fold(
        onSuccess = { paymentData ->
            continuePaymentFlow(paymentData)
        },
        onFailure = { throwable ->
            handleThrowable(throwable)
        }
    )
}
```

Start a payment by creating a Transaction object and using a paymentMethod from the paymentData. The amount is sent in cents:
```kotlin
val transaction = Transaction(
    transactionType = TransactionType.AUTH_CAPTURE,
    subscriptionType = SubscriptionType.INIT,
    merchantOrderId = merchantOrderID,
    merchantCustomerId = "5678",
    merchantContactUrl = URL("https://www.acquired.com"),
    merchantCustom1 = "custom1",
    merchantCustom2 = "custom2",
    merchantCustom3 = "custom3"
)

viewModelScope.launch {
    paymentGateway.pay(paymentMethod, transaction, 1234).fold(
        onSuccess = { orderData ->
            handleOrderData(orderData)
        },
        onFailure = {throwable ->
            handleThrowable(throwable)
        }
    )
}
```

Error handling?
-------------------
All known error is caught and put in a Failure, so trowables can be handled like this:
```kotlin
fun handleThrowable(throwable: Throwable) {
    when (throwable) {
        is Failure -> { handleFailure(throwable) }
        else -> {
            handleUnknownErrors(throwable)
        }
    }
}
```

General Failures
```kotlin
fun handleFailure(failure: Failure) {
    when(failure) {
        is CanceledByUserFailure -> { TODO("Handle error") }
        is DataFailure -> { TODO("Handle error") }
        is PaymentFailure -> { TODO("Handle error") }
        is NetworkFailure ->  { TODO("Handle error") }
        is PaymentAuthorizationFailure -> { handlePaymentAuthorizationFailure(failure) }
        is UnknownFailure -> { TODO("Handle error") }
    }
}
```

Specific Failures
```kotlin
fun handlePaymentAuthorizationFailure(failure: PaymentAuthorizationFailure) {
    when (failure) {
        is PaymentAuthorizationFailure.Blocked -> { TODO("Handle error") }
        is PaymentAuthorizationFailure.Declined -> { TODO("Handle error") }
        is PaymentAuthorizationFailure.Error -> { TODO("Handle error") }
        is PaymentAuthorizationFailure.InvalidCode -> { TODO("Handle error") }
        is PaymentAuthorizationFailure.Pending -> { TODO("Handle error") }
        is PaymentAuthorizationFailure.Quarantined -> { TODO("Handle error") }
        is PaymentAuthorizationFailure.TdsFailure -> { TODO("Handle error") }
        is PaymentAuthorizationFailure.Unknown -> { TODO("Handle error") }
    }
}
```
