---
layout: page
title: "Builder"
permalink: /patterns/builder
---

Object has many params / complex setup?
Separate **construction** from **representation** and build step-by-step.

**Intent:** Encapsulate object construction logic in a separate Builder.  
**When:**
- Many optional parameters
- Complex / conditional setup
- You want a readable, fluent creation API

Kotlin note: named + default params often replace simple builders. Use Builder when invariants / steps matter, or for Java interop.

Example:
```kotlin
data class User(
    val id: String,
    val name: String,
    val email: String?,
    val isAdmin: Boolean,
    val tags: List<String>
)

class UserBuilder {
    private var id: String? = null
    private var name: String? = null
    private var email: String? = null
    private var isAdmin: Boolean = false
    private val tags: MutableList<String> = mutableListOf()

    fun id(id: String) = apply { this.id = id }
    fun name(name: String) = apply { this.name = name }
    fun email(email: String?) = apply { this.email = email }
    fun admin() = apply { this.isAdmin = true }
    fun addTag(tag: String) = apply { this.tags += tag }

    fun build(): User {
        val id = requireNotNull(id) { "id is required" }
        val name = requireNotNull(name) { "name is required" }

        return User(
            id = id,
            name = name,
            email = email,
            isAdmin = isAdmin,
            tags = tags.toList()
        )
    }
}

fun main() {
    val user = UserBuilder()
        .id("123")
        .name("Paul")
        .email("paul@example.com")
        .admin()
        .addTag("beta-tester")
        .addTag("newsletter")
        .build()

    println(user)
}
