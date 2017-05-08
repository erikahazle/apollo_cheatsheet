# Cheatsheet for using Apollo client with a React application


Currently, we are keeping all query and mutation templates in `app/graphql` directory.

Each resource has an individual file (eg. `users.js`) with all related fragments, queries and mutations being stored together.

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


Some queries will require you to specify a variables as in:


```
export const getUserById = gql`
  query ($id: Uuid!) {
    userById(id: $id) {
      ...UserDetails
    }
  }
  ${UserDetails}
`

```

## Making mutations


Eg:


```
export const deleteUserById = gql`
  mutation ($input: DeleteUserByIdInput!) {
    deleteUserById(input: $input) {
      user {
        id
      }
    }
  }
`
```

`input` variable in mutations will be of type specified in the graphql schema.

All mutations have to return something.


## Connecting Apollo to React components

Only containers should be wrapper in `graphql`:

```
const mapStateToProps = (state) => {}
const mapDispatchToProps = (dispatch) => {}

const withQueriesAndMutations = graphql(getUserById, {
    name: 'getUser', options: (props) => ({
    variables: { id: props.params.id }})
  })(graphql(updateUserById, {
    props({ ownProps, mutate }) {
      return {
        updateUser({variables}) {
          return mutate({
            variables,
            updateQueries: {
              getUsers: (prev, { mutationResult: { data: { updateUserById } } }) => {
                if (variables.input.userPatch.userState === 'DELETED') {
                  const deletedUserIndex = _.findIndex(prev.allUsers.nodes, {
                    'id': updateUserById.user.id
                  })

                  return update(prev, {
                    allUsers: {
                      nodes: {
                        $splice: [[deletedUserIndex, 1]]
                      }
                    }
                  })

                }
              }
            }
          })
        }
      }
    }
  })(Component)
)

const Container = connect(
  mapStateToProps,
  mapDispatchToProps
)(withQueriesAndMutations)

export default Container

```

We're currently using Apollo client and Redux store separately, so we wrap components with `graphql` queries and only then connect to Redux store through `connect`.




## Keeping UI updated (removing and adding from/to lists)


## Immutability helper for `updateQueries`





