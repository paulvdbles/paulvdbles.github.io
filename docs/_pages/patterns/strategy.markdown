---
layout: page
title: "Strategy"
permalink: /patterns/strategy
---
Class does something specific in different ways?
Extract all logic into separate classes (strategies)

The processor should not have conditional logic (don't select which strategy to use), instead inject the right strategy

Interface:
``` 
interface DiscountStrategy {
    fun applyDiscount(amount: Double): Double
}

``` 
Strategies:
``` 
class NoDiscount : DiscountStrategy {
    override fun applyDiscount(amount: Double): Double = amount
}

class PercentageDiscount(
    private val percent: Double // e.g. 10.0 = 10%
) : DiscountStrategy {
    override fun applyDiscount(amount: Double): Double =
        amount * (1 - percent / 100.0)
}

class FixedAmountDiscount(
    private val discount: Double
) : DiscountStrategy {
    override fun applyDiscount(amount: Double): Double =
        (amount - discount).coerceAtLeast(0.0)
}

``` 

Usage in a service:
``` 
class CheckoutService(
    private var discountStrategy: DiscountStrategy
) {

    fun setDiscountStrategy(strategy: DiscountStrategy) {
        this.discountStrategy = strategy
    }

    fun checkout(itemsTotal: Double): Double {
        return discountStrategy.applyDiscount(itemsTotal)
    }
}

fun main() {
    val checkout = CheckoutService(NoDiscount())

    println(checkout.checkout(100.0)) // 100.0

    checkout.setDiscountStrategy(PercentageDiscount(20.0))
    println(checkout.checkout(100.0)) // 80.0

    checkout.setDiscountStrategy(FixedAmountDiscount(15.0))
    println(checkout.checkout(100.0)) // 85.0
}

``` 