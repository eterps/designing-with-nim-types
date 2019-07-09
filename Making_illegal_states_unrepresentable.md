# Designing with Nim types: Making illegal states unrepresentable

This is a translation from F# to Nim of [Designing with types: Introduction](https://fsharpforfunandprofit.com/posts/designing-with-types-intro/).
If Nim provides a better (or alternative) way to implement these techniques, [add an issue](https://github.com/eterps/designing-with-nim-types/issues?utf8=%E2%9C%93&q=is%3Aissue) or propose changes in a pull-request.

## Encoding business logic in types

### Making illegal states unrepresentable

WORK IN PROGRESS:

```nim
type
  PersonalName = object
    firstName: string
    middleInitial: Option[string]
    lastName: string

  EmailAddress = distinct string

  EmailContactInfo = object
    emailAddress: EmailAddress
    isEmailVerfied: bool

  ZipCode = distinct string
  StateCode = distinct string

  PostalAddress = object
    address1: string
    address2: string
    city: string
    state: StateCode
    zip: ZipCode

  PostalContactInfo = object
    address: PostalAddress
    isAddressValid: bool

variant ContactInfo:
  EmailOnly(email: EmailContactInfo)
  PostOnly(post: PostalContactInfo)
  EmailAndPost(emailAndPost: (EmailContactInfo, PostalContactInfo))

type Contact = object
  name: PersonalName
  contactInfo: ContactInfo
```
