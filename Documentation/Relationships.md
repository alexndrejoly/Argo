# Decoding Relationships

It's very common to have models that relate to other models. When all your
models conform to `Decodable`, Argo makes it really easy to populate those
relationships. Let's look at a `Post` and `Comment` model and how they relate.

Our server is sending us the JSON for the `Post` model:

```
{
  "author": "Gob Bluth",
  "text": "I've made a huge mistake."
}
```

Our `Post` model will then be:

```swift
struct Post {
  let author: String
  let text: String
}
```

And, our implementation of `Decodable` for `Post` looks like this:

```swift
extension Post: Decodable {
  static func decode(j: JSON) -> Decoded<Post> {
    return curry(self.init)
      <^> j <| "author"
      <*> j <| "text"
  }
}
```

Great! Now we can decode JSON into `Post` models. However, let's be real, we
can't have posts without comments! Comments are like 90% of the fun on the
internet.

Most likely the JSON will contain an embedded array of `Comment` models:

```
{
  "author": "Lindsay",
  "text": "I have the afternoon free.",
  "comments": [
    {
      "author": "Lucille",
      "text": "Really? Did 'nothing' cancel?"
    }
  ]
}
```

So then `Comment` will look like:

```swift
struct Comment {
  let author: String
  let text: String
}

extension Comment: Decodable {
  static func decode(j: JSON) -> Decoded<Comment> {
    return curry(self.init)
      <^> j <| "author"
      <*> j <| "text"
  }
}
```

Now, we can add an array of comments to our `Post` model:

```swift
struct Post {
  let author: String
  let text: String
  let comments: [Comment]
}

extension Post: Decodable {
  static func decode(j: JSON) -> Decoded<Post> {
    return curry(self.init)
      <^> j <| "author"
      <*> j <| "text"
      <*> j <|| "comments"
  }
}
```

We added `comments` as a property on our `Post` model. Then we added a line to
decode the comments from the JSON. Notice how we use `<||` instead of `<|` with
`comments` because it is an _Array_.

Storing the name of the author with a post or comment isn't very flexible.
What we really want to do is tie posts and comments to users. If we use the
`User` struct from [Basic Usage], we can simply change the `author` property
from `String` to `User`. No joke! Take a look:

[Basic Usage]: Basic-Usage.md

First, our JSON will contain the embedded user models like so:

```
{
  "author": {
    "id": 53,
    "name": "Lindsay"
  },
  "text": "I have the afternoon free.",
  "comments": [
    {
      "author": {
        "id": 1,
        "name": "Lucille"
      },
      "text": "Really? Did 'nothing' cancel?"
    }
  ]
}
```

Then, we can add the `User` model to `Post` and `Comment`:

```swift
struct User {
  let id: Int
  let name: String
}

extension User: Decodable {
  static func decode(j: JSON) -> Decoded<User> {
    return curry(User.init)
      <^> j <| "id"
      <*> j <| "name"
  }
}

struct Post {
  let author: User
  let text: String
  let comments: [Comment]
}

extension Post: Decodable {
  static func decode(j: JSON) -> Decoded<Post> {
    return curry(self.init)
      <^> j <| "author"
      <*> j <| "text"
      <*> j <|| "comments"
  }
}

struct Comment {
  let author: User
  let text: String
}

extension Comment: Decodable {
  static func decode(j: JSON) -> Decoded<Comment> {
    return curry(self.init)
      <^> j <| "author"
      <*> j <| "text"
  }
}
```

That's it! "How does this work?", you ask? Well, Argo is smart enough to know it can decode
anything that conforms to `Decodable` because internally, Argo is simply calling
each type's `decode` function. In this example, `Post`, `Comment`, and `User`
all conform to `Decodable` so Argo looks at those types the same way it looks at
`String` or `Int`.
