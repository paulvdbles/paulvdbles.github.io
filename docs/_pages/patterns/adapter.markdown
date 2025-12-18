---
layout: page
title: "Adapter"
permalink: /patterns/adapter
---

Need to use a class with the “wrong” interface?  
Wrap it in an **Adapter** so it matches what your code expects.

**Intent:** Convert one interface into another that clients expect.  
**When:**
- You integrate a library / legacy class with an incompatible API
- You can’t (or don’t want to) change the existing class
- You want to isolate 3rd-party/legacy details

**Type:** Structural

Example: adapting a legacy payment client to your own interface.

```kotlin
// Your target interface (what the app expects)
interface PaymentProcessor {
    fun pay(amountInCents: Long): Boolean
}

// Third-party / legacy API you can't change
class LegacyPaymentGateway {
    fun makePayment(amountInDollars: Double): Int {
        println("Paying $$amountInDollars via legacy gateway")
        return 0 // 0 = success, other = error
    }
}

// Adapter: wraps LegacyPaymentGateway and exposes PaymentProcessor
class LegacyPaymentAdapter(
    private val gateway: LegacyPaymentGateway
) : PaymentProcessor {

    override fun pay(amountInCents: Long): Boolean {
        val dollars = amountInCents / 100.0
        val resultCode = gateway.makePayment(dollars)
        return resultCode == 0
    }
}

// Usage
fun main() {
    val gateway = LegacyPaymentGateway()
    val processor: PaymentProcessor = LegacyPaymentAdapter(gateway)

    val success = processor.pay(12_34) // €12.34
    println("Payment success: $success")
}
