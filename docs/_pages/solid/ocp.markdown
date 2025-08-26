---
layout: page
title: "Open/Closed Principle"
permalink: /solid/ocp
---
## Definition 
You should be able to extend the behavior of a system without having to modify that system.

### Open
A class is open when you can extend it (produce subclass and add new methods/fields, override behavior)

### Closed
A class is closed for modification when you shouldn't need to change its existing source code once it's been written, tested, and deployed. The class is ready to be used by other classes.

## Example
    // Base class is closed for modification but open for extension
    abstract class Employee {
        abstract val baseSalary: Double
        
        // Base behavior that can be extended
        open fun calculateSalary(): Double {
            return baseSalary
        }
        
        // Template method that uses the overrideable calculateSalary()
        fun generatePayslip(): String {
            val salary = calculateSalary()
            return "Monthly payment: $$salary"
        }
    }

    // Extend through inheritance without modifying the base class
    class Developer(override val baseSalary: Double) : Employee() {
        override fun calculateSalary(): Double {
            val bonus = baseSalary * 0.10 // 10% coding bonus
            return baseSalary + bonus
        }
    }

    class Manager(override val baseSalary: Double) : Employee() {
        override fun calculateSalary(): Double {
            val bonus = baseSalary * 0.25 // 25% management bonus
            val stockOptions = 5000.0
            return baseSalary + bonus + stockOptions
        }
    }

    class Intern(override val baseSalary: Double) : Employee() {
        // Uses base implementation - no override needed
    }

    // New employee type - extending without modifying existing classes
    class SalesRep(
        override val baseSalary: Double,
        private val salesCommission: Double
    ) : Employee() {
        override fun calculateSalary(): Double {
            return baseSalary + salesCommission
        }
    }