# Designing with Nim types: Making illegal states unrepresentable

This is a translation from F# to Nim of [Designing with types: Introduction](https://fsharpforfunandprofit.com/posts/designing-with-types-intro/).
If Nim provides a better (or alternative) way to implement these techniques, [add an issue](https://github.com/eterps/designing-with-nim-types/issues?utf8=%E2%9C%93&q=is%3Aissue) or propose changes in a pull-request.

## Encoding business logic in types

```nim
type Contact =
  name: Name
  emailContactInfo: EmailContactInfo
  postalContactInfo: PostalContactInfo
```

```nim
type Contact =
  name: Name
  emailContactInfo: Option[EmailContactInfo]
  postalContactInfo: Option[PostalContactInfo]
```

### Making illegal states unrepresentable

```nim
proc `==` *(a, b: EmailAddress): bool {.borrow.}
proc `==` *(a, b: ZipCode): bool {.borrow.}
proc `==` *(a, b: StateCode): bool {.borrow.}

variant ContactInfo:
  EmailOnly(email: EmailContactInfo)
  PostOnly(post: PostalContactInfo)
  EmailAndPost(emailAndPost: (EmailContactInfo, PostalContactInfo))

type Contact = object
  name: PersonalName
  contactInfo: ContactInfo
```

This Nim translation uses the `variant` macro from a library called patty [to create a union type](https://github.com/andreaferretti/patty#constructing-variant-objects).

Also note the `==` procedure declarations, `distinct` types in Nim do not have the procedures of its base type available to them.
[See here](https://nim-by-example.github.io/types/distinct/) for an explanation.
In this case they are needed to compare the individual cases of the variant type.

### Constructing a ContactInfo

```nim
proc contactFromEmail(name: PersonalName, emailStr: string): Option[Contact] =
  let emailOpt = emailStr.createEmailAddress
  if emailOpt.isSome():
    let email = emailOpt.get()
    let emailContactInfo = EmailContactInfo(emailAddress: email, isEmailVerfied: false)
    let contactInfo = EmailOnly(emailContactInfo)
    some(Contact(name: name, contactInfo: contactInfo))
  else:
    none(Contact)

let name = PersonalName(firstName: "A", middleInitial: none(string), lastName: "Smith")
let contactOpt = contactFromEmail(name, "abc@example.com")
```

### Updating a ContactInfo

```nim
proc updatePostalAddress(contact: Contact, newPostalAddress: PostalContactInfo): Contact =
  var newContactInfo: ContactInfo
  match contact.contactInfo:
    EmailOnly(email):
      newContactInfo = EmailAndPost((email, newPostalAddress))
    PostOnly(_):
      newContactInfo = PostOnly(newPostalAddress)
    EmailAndPost(email_and_post):
      let (email, _) = email_and_post
      newContactInfo = EmailAndPost((email, newPostalAddress))
  # make a new contact
  Contact(name: contact.name, contactInfo: newContactInfo)

let contact = contactOpt.get
let newPostalAddress = PostalContactInfo(
  address: PostalAddress(
    address1: "123 Main",
    address2: "",
    city: "Beverly Hills",
    state: "CA".StateCode,
    zip: "92710".ZipCode),
  isAddressValid: false)
let newContact = updatePostalAddress(contact, newPostalAddress)
```
