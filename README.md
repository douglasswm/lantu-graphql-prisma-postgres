# Lantu GraphQL Prisma Server with PostgreSQL

Implementation of a **GraphQL server with an email-password-based authentication workflow and authentication rules**, based on Prisma, [graphql-yoga](https://github.com/prisma/graphql-yoga), [graphql-shield](https://github.com/maticzav/graphql-shield) & [GraphQL Nexus](https://graphql-nexus.com/).

## How to use

### 1. Download example & install dependencies

Clone the repository

```
git clone https://github.com/douglasswm/lantu-graphql-prisma-postgres.git
```

Install Node dependencies:

```
npm install
```

### 2. Install the Prisma CLI

To run, you need the Prisma CLI. Please install it via NPM or [using another method](https://www.prisma.io/docs/prisma-cli-and-configuration/using-the-prisma-cli-alx4/#installation):

```
npm install -g prisma
```

### 3. Set up database & deploy Prisma datamodel

Run Prisma locally via Docker:

1. Ensure you have Docker installed on your machine. If not, you can get it from [here](https://store.docker.com/search?offering=community&type=edition).
2. Run `docker-compose up -d`

To set up database, run:

```
prisma deploy
```

You can now use [Prisma Admin](https://www.prisma.io/docs/prisma-admin/overview-el3e/) to view and edit your data by appending `/_admin` to your Prisma endpoint.

### 4. Start the GraphQL server

Launch your GraphQL server with this command:

```
npm run dev
```

### 5. Using the GraphQL API

The schema that specifies the API operations of your GraphQL server is defined in [`./src/schema.graphql`](./src/schema.graphql). Below are a number of operations that you can send to the API using the GraphQL Playground.

Feel free to adjust any operation by adding or removing fields. The GraphQL Playground helps you with its auto-completion and query validation features.

#### Retrieve all published posts and their authors

```graphql
query {
  feed {
    id
    title
    content
    published
    author {
      id
      name
      email
    }
  }
}
```

<Details><Summary><strong>See more API operations</strong></Summary>

#### Register a new user

You can send the following mutation in the Playground to sign up a new user and retrieve an authentication token for them:

```graphql
mutation {
  signup(name: "Alice", email: "alice@prisma.io", password: "graphql") {
    token
  }
}
```

#### Log in an existing user

This mutation will log in an existing user by requesting a new authentication token for them:

```graphql
mutation {
  login(email: "alice@prisma.io", password: "graphql") {
    token
  }
}
```

#### Check whether a user is currently logged in with the `me` query

For this query, you need to make sure a valid authentication token is sent along with the `Bearer`-prefix in the `Authorization` header of the request:

```json
{
  "Authorization": "Bearer __YOUR_TOKEN__"
}
```

With a real token, this looks similar to this:

```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiJjanAydHJyczFmczE1MGEwM3kxaWl6c285IiwiaWF0IjoxNTQzNTA5NjY1fQ.Vx6ad6DuXA0FSQVyaIngOHYVzjKwbwq45flQslnqX04"
}
```

Inside the Playground, you can set HTTP headers in the bottom-left corner:

![](https://imgur.com/ToRcCTj.png)

Once you've set the header, you can send the following query to check whether the token is valid:

```graphql
{
  me {
    id
    name
    email
  }
}
```

#### Create a new draft

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground.

```graphql
mutation {
  createDraft(
    title: "Join the Prisma Slack"
    content: "https://slack.prisma.io"
  ) {
    id
    published
  }
}
```

#### Publish an existing draft

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground. The authentication token must belong to the user who created the post.

```graphql
mutation {
  publish(id: "__POST_ID__") {
    id
    published
  }
}
```

> **Note**: You need to replace the `__POST_ID__`-placeholder with an actual `id` from a `Post` item. You can find one e.g. using the `filterPosts`-query.

#### Search for posts with a specific title or content

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground. 

```graphql
{
  filterPosts(searchString: "graphql") {
    id
    title
    content
    published 
    author {
      id
      name
      email
    }
  }
}
```

#### Retrieve a single post

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground. 

```graphql
{
  post(id: "__POST_ID__") {
    id
    title
    content
    published
    author {
      id
      name
      email
    }
  }
}
```

> **Note**: You need to replace the `__POST_ID__`-placeholder with an actual `id` from a `Post` item. You can find one e.g. using the `filterPosts`-query.

#### Delete a post

You need to be logged in for this query to work, i.e. an authentication token that was retrieved through a `signup` or `login` mutation needs to be added to the `Authorization` header in the GraphQL Playground. The authentication token must belong to the user who created the post.

```graphql
mutation {
  deletePost(id: "__POST_ID__") {
    id
  }
}
```

> **Note**: You need to replace the `__POST_ID__`-placeholder with an actual `id` from a `Post` item. You can find one e.g. using the `filterPosts`-query.

</Details>

### 6. Changing the GraphQL schema

To make changes to the GraphQL schema, you need to manipulate the [`Query`](./src/resolvers/Query.ts) and [`Mutation`](./src/resolvers/Mutation.ts) types. 

Note that the [`start`](./package.json#L6) script also starts a development server that automatically updates your schema every time you save a file. This way, the auto-generated [GraphQL schema](./src/generated/schema.graphql) updates whenever you make changes in to the `Query` or `Mutation` types inside your TypeScript code.

## Commit Guidelines
Mostly reasonable guidelines for commit messages.

## Commit Message Format
Commit messages consist of a [message](#message), [description](#description)
and [related](#related).

```
<message>

<description>

<related>
```

A [message](#message) consists of a [type](#type), [scope](#scope), and
[subject](#subject).

No line should be longer than 80 characters long, for optimal github viewing.

## Message
```
<type>(<scope>): <subject>
```

Examples: `refactor(plans#show): remove 1 of 4 templating languages`

### Type
* `feat`: New feature
* `fix`: Bug fix
* `format`: Change not affecting the **meaning** of code (white-space, formatting, etc)
* `docs`: Documentation-**only** change
* `style`: Stylesheet-**only** change
* `refactor`: Change that neither fixes a bug or adds a feature
* `perf`: Change that improves performance
* `test`: Addition of a missing test
* `chore`: Changes to the build process or auxiliary tools and libraries

### Scope
The scope could be anything specifying the site of the commit change. For
example `plans#show`, `PlanPerson`, `LiveChatMessageList`, `.tab-list`, etc...

### Subject
The subject contains a succinct description of the change:

* use the imperative, present tense: "change" not "changed" nor "changes"
* don't use capitalize first letters
* don't use trailing punctuation (.)

### Description
Just as in the [subject](#subject), use the imperative, present tense: "change"
not "changed" nor "changes" The body should include the motivation for the
change.

### Related
The footer should contain any information about **Breaking Changes** and is the
place to reference Trello cards, or GitHub issues that the commit **Closes**.

### Attribution
This guide is ~~pirated from~~ inspired by Angular's excellent
[CONTRIBUTING.md](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#-git-commit-guidelines).

A detailed explanation can be found in this [document][commit-message-format].

[commit-message-format]: https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#

## `fixup` commits

Your stuff's on staging but you're making small, progressive tweeks. `fixup` is your friend.

Fixup commits are associated with commit you've already made. They allow you to make commits without inserting additional messages. They can also be automatically squashed, for those civilized developers.

Here's how it works. `commit` takes `--fixup` as on option. When used, you'll have to provide the SHA of the commit your is fixing up. Like so:

```bash
git commit --fixup a9b8c7

# aliases also work but get confusing after your first fix

git commit --fixup head
```

You probably don't memorize your recent commit SHAs. You can use the syntax below as a backword search. This fixup commit will attach to the most recent commit, where "style" is in the message:

```bash
git commit --fixup :/style
```
