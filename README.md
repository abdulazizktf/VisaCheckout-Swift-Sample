# VisaCheckout-Swift-Sample
Hands on sample for integrating VCO using the Judopay Swift SDK

## Requirements
- Xcode 9
- Swift 4
- JudoKit 2.6.21+
- Visa Checkout SDK 5.5.2+

## Getting started

#### 1. Add the Visa Checkout SDK to the project
##### Cocoapods
Add the Visa Checkout pod to your `Podfile`

```ruby
  pod 'JudoKit', '~> 7.0.0'
  pod 'VisaCheckout', '~> 5.5.2-9.1'
```

Run `pod install`

##### Manually
Download the [Visa Checkout SDK](https://developer.visa.com/capabilities/visa_checkout/docs#adding_visa_checkout_to_a_mobile_application) and follow the instructions in the bundled documentation.

#### 2. Configure the Visa Checkout SDK
Add the VCO configuration in your `AppDelegate.swift` after importing the `VisaCheckoutSDK`

```swift
let profile = Profile(environment: .sandbox, apiKey: "your VCO api key")
/// An arbitrary example of some configuration details you can customize.
/// See the documentation/headers for `Profile`.
profile.datalevel = .full
profile.acceptedCardBrands = [.visa, .mastercard, .discover]

VisaCheckoutSDK.configure(profile: profile)

```

#### 3. Add the Visa Checkout button to your screen
- Copy the `VisaCheckoutButton.swift` into your project.
- Add a `View` to your `ViewController`
- Set the custom class of the view to `VisaCheckoutButton`

#### 4. Create a `PurchaseInfo` and set up the `VisaCheckoutButton`

```swift
let purchaseInfo = PurchaseInfo(total: 10.99, currency: .gbp)
purchaseInfo.reviewAction = .pay
purchaseInfo.promoCode = "PROMO1"
purchaseInfo.discount = CurrencyAmount(decimalNumber: 1.99)
purchaseInfo.orderId = orderId
checkoutButton.onCheckout(purchaseInfo: purchaseInfo, completion: visaCheckoutResultHandler)
```

#### 5. Handle the VCO response and pass on the result to Judo

```swift
private func visaCheckoutResultHandler(result: CheckoutResult) {
    switch result.statusCode {
    case .success:
        if let callId = result.callId, let encryptedKey = result.encryptedKey, let encryptedPaymentData = result.encryptedPaymentData {
            let amount = Amount(decimalNumber: 10.99, currency: .GBP)
            let reference = Reference(consumerRef: UUID().uuidString, paymentRef: orderId)
            let vcoResult = VCOResult(callId: callId, encryptedKey: encryptedKey, encryptedPaymentData: encryptedPaymentData)
            _ = try? judoKit
                .payment(judoId, amount: amount, reference: reference)
                .vcoResult(vcoResult)
                .completion(judoCompletionBlock)
        }
    case .userCancelled:
        print("Payment cancelled by the user")
    default:
        break
    }
}
```

#### 6. Handle the Judo callback to check the payment results

```swift
private func judoCompletionBlock(response: Response?, error: JudoError?) {
    if let response = response, response.items.count > 0, response.items[0].result == .Success {
        print("Payment successful!")
    } else {
        print("Oops. Something went wrong.")
    }
}
```
