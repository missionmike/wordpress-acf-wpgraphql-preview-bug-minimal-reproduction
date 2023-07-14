# Minimal Reproduction of Custom Fields in Preview Mode Bug

## Setup

> Note: You'll need to have Docker installed to run this.

1. Clone this repository.
2. Open in VS Code (or other IDE), and run `./run init` from the project root.
3. After containers have started, run `./run mysql:import` to import the DB.
4. Be sure you can sign into wp-admin on http://localhost:8000/wp-admin with username/password: admin/admin
5. Disable the Classic Editor plugin. This issue does not happen when using the Classic Editor, but the plugin is included here for testing & debugging purposes.
6. Open Postman, and configure a `POST` request to http://localhost:8000/graphql and set the body to:

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

## Description of Issue

When the post content, title, or other standard WordPress fields are updated and previewed in WordPress via **Gutenberg editor**, the GraphQL response in preview mode will return `null` for associated custom fields.

For example, here is a response for preview of an already-published post. Note how the `mainArt` and `customFields` values are populated:

```
    "data": {
        "post": {
            "title": "Hello world!",
            "content": "\n<p>First, preview this post without making any changes. Then check the response in Postman. You should see something like:</p>\n\n\n\n<pre class=\"wp-block-code\"><code>    \"data\": {\n        \"post\": {\n            \"title\": \"Hello world!\",\n            \"content\": \"... omitted for brevity ...\",\n            \"status\": \"inherit\",\n            \"databaseId\": 30,\n            \"slug\": \"1-revision-v1\",\n            \"uri\": \"/hello-world/?preview=true\",\n            \"link\": \"http://localhost:8000/hello-world/?preview=true\",\n            \"date\": \"2023-07-13T19:41:38\",\n            \"mainArt\": {\n                \"mediaType\": \"image\",\n                \"image\": {\n                    \"sourceUrl\": \"http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f.jpg\",\n                    \"srcSet\": \"http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f.jpg 976w, http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f-300x261.jpg 300w, http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f-768x669.jpg 768w\",\n                    \"sizes\": \"(max-width: 976px) 100vw, 976px\",\n                    \"caption\": null\n                },\n                \"video\": null\n            },\n            \"customFields\": {\n                \"customTextField\": \"Change the body copy, then preview, but don't save. This value will be null in Postman.\"\n            }\n        }\n    },</code></pre>\n\n\n\n<p><strong>Change this body copy or title, then click &#8220;preview&#8221; to preview this post.</strong></p>\n\n\n\n<p>Then go to Postman and check the query response for the page. You should see something like:</p>\n\n\n\n<pre class=\"wp-block-code\"><code>    \"data\": {\n        \"post\": {\n            \"title\": \"Hello world!\",\n            \"content\": \"\\n&lt;p>Welcome to WordPress. This is your first post. Edit or delete it, then start writing!!!&lt;/p>\\n\",\n            \"status\": \"inherit\",\n            \"databaseId\": 31,\n            \"slug\": \"1-autosave-v1\",\n            \"uri\": \"/hello-world/?preview=true\",\n            \"link\": \"http://localhost:8000/hello-world/?preview=true\",\n            \"date\": \"2023-07-13T19:39:16\",\n            \"mainArt\": {\n                \"mediaType\": \"image\",\n                \"image\": null,\n                \"video\": null\n            },\n            \"customFields\": {\n                \"customTextField\": null\n            }\n        }\n    },</code></pre>\n\n\n\n<p>Note the <code>null</code> values for the <code>mainArt</code> and <code>customFields</code> group values. The preview mode does not seem to pull these in.</p>\n",
            "status": "inherit",
            "databaseId": 32,
            "slug": "1-revision-v1",
            "uri": "/hello-world/?preview=true",
            "link": "http://localhost:8000/hello-world/?preview=true",
            "date": "2023-07-13T19:51:37",
            "mainArt": {
                "mediaType": "image",
                "image": {
                    "sourceUrl": "http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f.jpg",
                    "srcSet": "http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f.jpg 976w, http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f-300x261.jpg 300w, http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f-768x669.jpg 768w",
                    "sizes": "(max-width: 976px) 100vw, 976px",
                    "caption": null
                },
                "video": null
            },
            "customFields": {
                "customTextField": "Change the body copy, then preview, but don't save. This value will be null in Postman."
            }
        }
    }
```

Now, I'll update some text content in the title, for testing. Then will preview again. Then, the Postman response is:

```
    "data": {
        "post": {
            "title": "Hello Universe!",
            "content": "\n<p>First, preview this post without making any changes. Then check the response in Postman. You should see something like:</p>\n\n\n\n<pre class=\"wp-block-code\"><code>    \"data\": {\n        \"post\": {\n            \"title\": \"Hello world!\",\n            \"content\": \"... omitted for brevity ...\",\n            \"status\": \"inherit\",\n            \"databaseId\": 30,\n            \"slug\": \"1-revision-v1\",\n            \"uri\": \"/hello-world/?preview=true\",\n            \"link\": \"http://localhost:8000/hello-world/?preview=true\",\n            \"date\": \"2023-07-13T19:41:38\",\n            \"mainArt\": {\n                \"mediaType\": \"image\",\n                \"image\": {\n                    \"sourceUrl\": \"http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f.jpg\",\n                    \"srcSet\": \"http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f.jpg 976w, http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f-300x261.jpg 300w, http://localhost:8000/wp-content/uploads/2023/07/91408619_55df76d5-2245-41c1-8031-07a4da3f313f-768x669.jpg 768w\",\n                    \"sizes\": \"(max-width: 976px) 100vw, 976px\",\n                    \"caption\": null\n                },\n                \"video\": null\n            },\n            \"customFields\": {\n                \"customTextField\": \"Change the body copy, then preview, but don't save. This value will be null in Postman.\"\n            }\n        }\n    },</code></pre>\n\n\n\n<p><strong>Change this body copy or title, then click &#8220;preview&#8221; to preview this post.</strong></p>\n\n\n\n<p>Then go to Postman and check the query response for the page. You should see something like:</p>\n\n\n\n<pre class=\"wp-block-code\"><code>    \"data\": {\n        \"post\": {\n            \"title\": \"Hello world!\",\n            \"content\": \"\\n&lt;p&gt;Welcome to WordPress. This is your first post. Edit or delete it, then start writing!!!&lt;/p&gt;\\n\",\n            \"status\": \"inherit\",\n            \"databaseId\": 31,\n            \"slug\": \"1-autosave-v1\",\n            \"uri\": \"/hello-world/?preview=true\",\n            \"link\": \"http://localhost:8000/hello-world/?preview=true\",\n            \"date\": \"2023-07-13T19:39:16\",\n            \"mainArt\": {\n                \"mediaType\": \"image\",\n                \"image\": null,\n                \"video\": null\n            },\n            \"customFields\": {\n                \"customTextField\": null\n            }\n        }\n    },</code></pre>\n\n\n\n<p>Note the <code>null</code> values for the <code>mainArt</code> and <code>customFields</code> group values. The preview mode does not seem to pull these in.</p>\n",
            "status": "inherit",
            "databaseId": 33,
            "slug": "1-autosave-v1",
            "uri": "/hello-world/?preview=true",
            "link": "http://localhost:8000/hello-world/?preview=true",
            "date": "2023-07-13T20:09:28",
            "mainArt": {
                "mediaType": "image",
                "image": null,
                "video": null
            },
            "customFields": {
                "customTextField": null
            }
        }
    },
```

Note how the title preview is correct, but the `mainArt` and `customFields` values are `null`.
