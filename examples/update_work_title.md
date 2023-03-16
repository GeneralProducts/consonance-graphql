# Description

Update a work to amend its title

## Explanation

This mutation updates the work level title is changed on a specific work, specified by its work ID. The results are then displayed as part of the query.

```gql
mutation {
  updateWork(input: {
    id: [WORK_ID],
    title: "Your New Title"
  }) {
    work {
      id
      title
    }
  }
}
```
