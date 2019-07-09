# Designing with Nim types: Single case union types

This is a translation from F# to Nim of [Designing with types: Introduction](https://fsharpforfunandprofit.com/posts/designing-with-types-intro/).
If Nim provides a better (or alternative) way to implement these techniques, [add an issue](https://github.com/eterps/designing-with-nim-types/issues?utf8=%E2%9C%93&q=is%3Aissue) or propose changes in a pull-request.

## Adding meaning to primitive types

### Wrapping primitive types

```nim
type
  EmailAddress = distinct string

# using constructor as a function
discard "a".EmailAddress
discard @["a", "b", "c"].mapIt(it.EmailAddress)

# inline deconstruction
let a’ = "a".EmailAddress
let a’’ = a’.string

let addresses = @["a", "b", "c"].mapIt(it.EmailAddress)
let addresses’ = addresses.mapIt(it.string)

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

  Contact = object
    name: PersonalName
    emailContactInfo: EmailContactInfo
    postalContactInfo: PostalContactInfo
```

### Naming...

```nim
var f: proc(emailAddress: string): EmailAddress

let y = "test@me.com"
let x = y.EmailAddress
```

### Constructing...

```nim
proc createEmailAddress(s: string): Option[EmailAddress] =
  if s.match(re"^\S+@\S+\.\S+$"):
    some(s.EmailAddress)
  else:
    none(EmailAddress)

proc createStateCode(s: string): Option[StateCode] =
  let stateCodes = ["AZ", "CA", "NY"] # etc
  if stateCodes.any(x => x == s.toUpper):
    some(s.StateCode)
  else:
    none(StateCode)

discard createStateCode("CA")

discard createEmailAddress("a@example.com")
```

### Handling invalid input in a constructor

```nim
type CreationResult = Result[EmailAddress, string]
proc createEmailAddress2(s: string): CreationResult =
  if s.match(re"^\S+@\S+\.\S+$"):
    CreationResult.ok s.EmailAddress
  else:
    CreationResult.err "Email address must contain an @ sign"

discard createEmailAddress2("example.com")

proc createEmailAddressWithContinuations(success, failure: auto, s: string) =
  if s.match(re"^\S+@\S+\.\S+$"):
    success(s.EmailAddress)
  else:
    failure("Email address must contain an @ sign")

proc success(s: EmailAddress) = echo "success creating email " & s.string
proc failure(msg: string) = echo "error creating email: " & msg
createEmailAddressWithContinuations(success, failure, "example.com")
createEmailAddressWithContinuations(success, failure, "x@example.com")
```

Next: [Designing with Nim types: Making illegal states unrepresentable](Making_illegal_states_unrepresentable.md)
