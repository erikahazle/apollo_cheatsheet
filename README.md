# Cheatsheet for using Apollo client with a React application


Currently, we are keeping all query and mutation templates in `app/graphql` directory.

Each resource has an individual file (eg. `campaigns.js`) with all fragments, queries and mutations being stored together.

The use of fragments is encouraged to limit boilerplate for queries and mutations.


## Making queries

Eg:


```
const UserDetails = gql`
  fragment UserDetails on User {
    id
    name
  }
`

export const getUsers = gql`
  query getUsers {
    allUsers(condition: {userState: ACTIVE}) {
      nodes {
        ...UserDetails
      }
    }
  }
  ${UserDetails}
`
```

Passing in condition object will allow you to filter data that comes from the API.

Name your queries to call them in update queries for updating lists.


## Making mutations



## Keeping UI updated (removing and adding from/to lists)


