---
layout: page
title: "Single Responsibility Principle"
permalink: /solid/srp
---

## Definition 
A module should have one, and only one, reason to change

When SRP is applied correctly changes to a module should have minimal impact on other modules.

When we design a system we should:
* learn about the actors
* identify responsibilities
* allocate responsibilities to modules

## Example
### Without SRP
    // This class has multiple responsibilities - it's doing too much!
    class Employee {
        var name: String = ""
        var email: String = ""
        var hourlyRate: Double = 0.0
        var hoursWorked: Int = 0
        
        // Responsibility 1: Managing employee data
        fun updateEmployeeInfo(name: String, email: String) {
            this.name = name
            this.email = email
        }
        
        // Responsibility 2: Calculating salary
        fun calculateSalary(): Double {
            return hourlyRate * hoursWorked
        }
        
        // Responsibility 3: Saving to database
        fun saveToDatabase() {
            println("Connecting to database...")
            println("Saving employee $name to database")
            // Database logic here
        }
        
        // Responsibility 4: Sending emails
        fun sendPayslipEmail() {
            println("Sending email to $email")
            println("Your salary is: ${calculateSalary()}")
            // Email sending logic here
        }
    }

### With SRP
    // Each class has a single, well-defined responsibility

    // Responsibility: Representing employee data
    data class Employee(
        val name: String,
        val email: String,
        val hourlyRate: Double,
        val hoursWorked: Int
    )

    // Responsibility: Calculating employee salary
    class SalaryCalculator {
        fun calculate(employee: Employee): Double {
            return employee.hourlyRate * employee.hoursWorked
        }
        
        fun calculateWithBonus(employee: Employee, bonusPercentage: Double): Double {
            val baseSalary = calculate(employee)
            return baseSalary * (1 + bonusPercentage / 100)
        }
    }

    // Responsibility: Persisting employee data
    class EmployeeRepository {
        fun save(employee: Employee) {
            println("Connecting to database...")
            println("Saving employee ${employee.name} to database")
            // Database logic here
        }
        
        fun findById(id: Int): Employee? {
            // Database query logic
            println("Fetching employee with ID: $id")
            return null // Simplified for example
        }
    }

    // Responsibility: Handling email notifications
    class EmailService {
        fun sendPayslipEmail(employee: Employee, salary: Double) {
            println("Sending email to ${employee.email}")
            println("Dear ${employee.name}, your salary is: $$salary")
            // Email sending logic here
        }
        
        fun sendWelcomeEmail(employee: Employee) {
            println("Sending welcome email to ${employee.email}")
            // Email logic here
        }
    }

    // Usage - composing the classes together
    fun main() {
        // Create employee
        val employee = Employee(
            name = "John Doe",
            email = "john@example.com",
            hourlyRate = 25.0,
            hoursWorked = 160
        )
        
        // Each class handles its own responsibility
        val calculator = SalaryCalculator()
        val repository = EmployeeRepository()
        val emailService = EmailService()
        
        // Calculate salary
        val salary = calculator.calculate(employee)
        val salaryWithBonus = calculator.calculateWithBonus(employee, 10.0)
        
        // Save to database
        repository.save(employee)
        
        // Send email
        emailService.sendPayslipEmail(employee, salaryWithBonus)
    }