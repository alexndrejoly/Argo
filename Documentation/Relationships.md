# Decoding Relationships

It's very common to have custom models that relate to other custom models.
When all your models conform to `Decodable`, Argo makes it really easy to
populate those relationships. Let's look at a `Post` and `Comment` model and
how they relate.

Our `Post` model will be very simple:

```swift
struct Post {
  let author: String
  let text: String
}
```

Then, our implemention of `Decodable` for `Post` looks like this:

```swift
extension Post: Decodable {
  static func decode(j: JSON) -> Decoded<Post> {
    return curry(self.init)
      <^> j <| "author"
      <*> j <| "text"
  }
}
```

And the JSON looks like this:

```
{
  "author": "Gob Bluth",
  "text": "I've made a huge mistake.."
}
```

Great! Now we can decode JSON into `Post` models. Let's be real, we can't have
posts without comments! Comments are like 90% of the fun on the internet.

So let's add a simple `Comment` model:

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

We added the array as a property on our `Post` model. Then we added a line to
decode the comments array from the JSON. Notice how we use `<||` instead of
`<|` with `comments` because it is an _Array_.

With the embedded comments array, the JSON could look like this:

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

Storing the name of the author with a post or comment isn't very flexible.
What we really want to do is tie posts and comments to users. If we use the
`User` struct from [Basic Usage], we can simply change the `author` property
from `String` to `User`.

[Basic Usage]: Basic-Usage.md

```swift
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

Now the JSON for a post could look like this:

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

Yep that's right, the only thing that changed was the type of `author`: it was
a `String` and now it's a `User`!

We can also create a convenience property to directly access the user's name
instead of having to compute it from the model later.

```swift
struct Post {
  let author: User
  let authorName: String
  let text: String
  let comments: [Comment]
}

extension Post: Decodable {
  static func decode(j: JSON) -> Decoded<Post> {
    return curry(self.init)
      <^> j <| "author"
      <*> j <| ["author", "name"]
      <*> j <| "text"
      <*> j <|| "comments"
  }
}

struct Comment {
  let author: User
  let authorName: String
  let text: String
}

extension Comment: Decodable {
  static func decode(j: JSON) -> Decoded<Comment> {
    return curry(self.init)
      <^> j <| "author"
      <*> j <| ["author", "name"]
      <*> j <| "text"
  }
}
```

Using an array of strings allows us to traverse embedded objects to get at the
value we want.
