# Designing with Nim types: Introduction

This is a translation from F# to Nim of [Designing with types: Introduction](https://fsharpforfunandprofit.com/posts/designing-with-types-intro/).
If Nim provides a better (or alternative) way to implement these techniques, [add an issue](https://github.com/eterps/designing-with-nim-types/issues?utf8=%E2%9C%93&q=is%3Aissue) or propose changes in a pull-request.

## Making design more transparent and improving correctness

### A basic example

```nim
type Contact = object
  firstName: string
  middleInitial: string
  lastName: string

  emailAddress: string
  isEmailVerfied: bool # true if ownership of email address is confirmed

  address1: string
  address2: string
  city: string
  state: string
  zip: string
  isAddressValid: bool # true if validated against address service
```

### Creating “atomic” types

```nim
type
  PersonalName = object
    firstName: string
    middleInitial: Option[string]
    lastName: string

  EmailContactInfo = object
    emailAddress: string
    isEmailVerfied: bool

  PostalAddress = object
    address1: string
    address2: string
    city: string
    state: string
    zip: string

  PostalContactInfo = object
    address: PostalAddress
    isAddressValid: bool

  Contact = object
    name: PersonalName
    emailContactInfo: EmailContactInfo
    postalContactInfo: PostalContactInfo
```

Next: [Designing with Nim types: Single case union types](Single case union types.md)
