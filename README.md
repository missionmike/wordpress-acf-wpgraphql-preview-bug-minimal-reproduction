# Minimal Reproduction of Custom Fields in Preview Mode Bug

## Setup

1. Clone this repository.
2. Open in VS Code (or other IDE), and run `./run init` from the project root.
3. After containers have started, run `./run mysql:import` to import the DB.
4. Sign into wp-admin on http://localhost:8000/wp-admin with username/password: admin/admin
5. Open Postman, and configure a `getPost` request with type of `POST`, and set the body to:

```
query getPost($id: ID!) {
  post(id: $id, idType: DATABASE_ID, asPreview: true) {
    title
    content
    status
    databaseId
    slug
    uri
    link
    date
    mainArt {
        mediaType
        image {
        sourceUrl(size: LARGE)
        srcSet(size: LARGE)
        sizes(size: LARGE)
        caption
        }
        video
    }
    customFields {
        customTextField
    }
  }
}
```

And for variables:

```
{
    "id": 1
}
```

6. In the `Authorization` section of Postman, set:

- Authorization Type: Basic Auth
- Username: `admin`
- Password: `AfOv c2NL aHL2 EVaL waic Geg3`

7. Test the request. You should get a response for the default "Hello world!" post.

See the post content for further instructions on testing the bug.
