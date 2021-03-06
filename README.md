# Redux Offline Apollo Link

_NOTE_ This is still TBD. Use at your own risk!!

This is a close fork of [apollo-upload-client](https://github.com/jaydenseric/apollo-upload-client), and acts as a drop in replacement.

I needed a way to get customized actions dispatched from apollo as network activity occured (request, success, failures).

This is closely tied with Redux Offline.

Redux Offline offers actions to be dispatched when a device is offline and go to a queue _(aka outbox)_.

[Read more about Redux Offline](https://github.com/redux-offline/redux-offline)

### Requirements

- `@redux-offline/redux-offline: ^2.0.0`
- `apollo-client: ^2.4.8`

### Installation

```js
import { ApolloClient } from "apollo-client";
import { reduxOfflineApolloLink } from "redux-offline-apollo-link";
import { store } from "../../store";

export const client = new ApolloClient({
  link: ApolloLink.from([
    reduxOfflineApolloLink(
      {
        uri: "http://your.graphql.server.com:3001/graphql"
      },
      store
    )
  ]),
  cache: new InMemoryCache()
});
```

### Usage in Components

In order to use the redux actions, you need to provide a `variables.[VARIABLE NAME]` to your `graphql` higher order component call.


The only *REQUIRED* variable to date is `actionType`





### Variables


- `actionType` *REQUIRED* - The name of the request action
- `actionCommitSuffix` - Suffix of the action type when a success occurs _Default: "COMMIT"_
- `actionRollbackSuffix` - Suffix of the action type when a rollback occurs _Default: "ROLLBACK"_
- `options` - Suffix of the action type when a rollback occurs _Default: "ROLLBACK"_
	- `payloadFormatter(payload)` Optional Function to format the response coming back from GraphQL on a successful retrieval. `payload` is provided as a parameter.  This function returns back the modified data.  This is a good place to run data normalizations
  - `errorsCheck(result)` Optional Function to give the client the choice for whether it should throw an error if errors are present on GraphQL response. It throws an error by default if function is not provided.
  - `parseAndHandleHttpResponse(operation, result)` Optional Function to give the client the choice for whether it should throw an error if there's an HTTP/GraphQL schema error present on GraphQL response. It throws an error by default if the function is not provided.
  - `skipOffline` Skips offline check returning error right away if connection can't be stablished.

---

### Examples


**graphql Query HoC Example**

```js
export default compose(
  graphql(
    gql`
      query uploads {
        uploads {
          id
          filename
          mimetype
          path
        }
      }
    `,
    {
      options: {
        variables: {
          actionType: "DEMO_QUERY",
          actionCommitSuffix: "COMMIT",
          actionRollbackSuffix: "ROLLBACK",
          options: {
            payloadFormatter(payload) {
              return doSomething({
                ...payload,
                extra: true,
              });
            }
          }
        }
      }
    }
  ),
  connect(state => ({}))
)(DemoQuery);
```

**graphql Mutation Example**

```js
export default compose(
  graphql(
    gql`
      mutation($file: Upload!) {
        singleUpload(file: $file) {
          id
          filename
          mimetype
          path
        }
      }
    `,
    {
      options: {
        variables: {
          actionType: "DEMO_MUTATION"
        }
      }
    }
  ),
  connect(state => ({
    images: state.images.images
  }))
)(Mutation);
```


**Direct Client call Example**

```js
const { client } = this.props;
return client
  .query({
    query,
    fetchPolicy: "no-cache",
    variables: {
      startDateTimeUtc,
      endDateTimeUtc,
      actionType: "TODAY_SCHEDULES_REQUEST",
      options: {
        payloadFormatter(payload: any) {
          return payload;
        }
      }
    }
  })
```

---






