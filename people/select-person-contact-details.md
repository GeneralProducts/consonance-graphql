# Select person contact details

## Explanation

Person IDs are retrieved for a single contact along with their relevant contact information.

```
query GetPersonContactInfo($personId: Int!) {
  person(id: $personId) {
    id
    emails {
      email
    }
    phones {
      number
    }
    webLinks {
      url
    }
  }
}

```
