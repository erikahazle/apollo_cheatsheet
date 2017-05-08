# Dummy's guide to using Apollo client with a React application


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

`input` variable in mutations is of type specified in the `graphql` schema.

All mutations have to return something.


## Connecting Apollo to React components

Only containers should be wrapper in `graphql` (`UserContainer.js`):

```
const mapStateToProps = (state) => {}
const mapDispatchToProps = (dispatch) => {}

const withQueriesAndMutations = graphql(getUserById, {
  name: 'getUser',
  options: (props) => ({
    variables: { id: props.params.id }
  })
})(Component)

const Container = connect(
  mapStateToProps,
  mapDispatchToProps
)(withQueriesAndMutations)

export default Container

```

We're currently using Apollo client and Redux store separately, so we wrap components with `graphql` queries and only then connect to Redux store through `connect`.

Calls to the API are made using `graphql` helper. First argument is the query/mutation (imported from `graphql` directory). The second argument is an object that configures the call.

Some useful configurations for a simple query/mutation:

 - name: is the name of the `prop` in which the query/mutation will be passed down to the Component
 - options: this object will contain variables that need to be passed into the query/mutation (eg. input)

## Keeping UI updated (removing and adding from/to lists)

Apollo updates lists within its store based on `id` (and `_id`), which means that to handle addition/removal to/from a list you will need to use `refetchQueries` or `updateQueries`

`updateQueries` are prefererable as they do not trigger extra calls to the API:

```
graphql(updateUserById, {
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
  })
```

`updateUser`, in this case, is the name of the prop that will be passed down to the component that is wrapped in `graphql`.

Fields within `updateQueries` refer to the names of the queries that need to be updated after mutation. In this example, data stored within `getUsers` will be affected by `updateUser` mutation.

## Immutability helper for `updateQueries`

Currently, we're using module 'immutability-helper'. For more information on available methods, see: https://facebook.github.io/react/docs/update.html

## Accessing Apollo generated props in your components

Example:

```
graphql(getUserById, {
  name: 'getUser',
  options: (props) => ({
    variables: { id: props.params.id }
  })
})
```

In the component this query will be accessed by `this.props.getUser` (name field within configuration).

This object will contain these useful fields:

- `loading`
- `error`
- `refetch` - this method will allow you to fetch the data from the server again
- `getUserById` - this field name is the same as the root query and, on successful load, will contain data fetched from your API


`loading` and `error` should be used to give feedback to the user.


