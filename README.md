# The Consonance GraphQL API

Welcome to the Consonance GraphQL API.

This is where we help you use the flexibility and efficiency of our GraphQL interface to get structured data from your single source of bibliographic truth.

## Why GraphQL?

Bibliographic metadata has a complex structure, in which works, products, series, people and organisation can contribute to different products on a work, and in which works are related to each other by being part of the same series, or having the same contributors.

This is difficult to capture in fixed data formats such as ONIX, which (quite appropriately) copes with this complexity by presenting product-oriented data, and also requires that a great deal of metadata is included which might not be necessary for every purpose.

GraphQL addresses these issues by treating the data as "types", such as `Work`, `Contact`, `Product`, `Prize`, and `Series`, which have "fields" such as `title`, `id`, and `name`, and which are connected together so that for an individual `Work` you can ask for its `products`, or its `contributions`. For each of a work's products you can request the ISBN, title, product form, and perhaps its USD library prices.

So GraphQL lets you programatically ask for exactly the data that you need, in two ways.

Firstly, it makes it possible to request a precise set of attributes, instead of needing to read data values that you don't need. This helps both Consonance and your client application by reducing the amount of data transferred, and the time taken to return it.

Secondly, in many cases it also lets you request the rows of data you want. For example you can get only the products for a particular type of work perhaps, or all of the prices but only for ebook products.

In some cases it offers different formats of the same data. For example, you may want ISBNs for inserting into a searchable system, so you want just the digits "9781234567890", or you might want to display them on a website in the official formatted pattern "978-0-01234-56789-0". This can be difficult to do, so you can ask the GraphQL API to do it for you.

Similarly you can retrieve product measurements in millimeters, centimeters, or inches, or weights in grams or ounces.

So the main benefit to using the GraphQL API is that it is excellent at dealing with complex data relations. If you want only works of a particular series, their ebook PDF products, their contributors, and the library prices for the products. To represent this in a CSV file would require a set of columns for the work, one set per product, another set for each contributor, and more columns for each price. Pretty soon your spreadsheet is at column HS, has column names like "product_1_price_4" and "contributor_3_last_name", and is unreadable.

In a GraphQL response these relations are expressed in a JSON response that is structured in the same way as your original query, and is readable by any popular programing language. Or even by a human being.

Lastly, one of the greatest benefits of GraphQL is that it is self-documenting. A client can issue a query against the GraphQL schema which we have defined for Consonance, and retrieve a result that describes the types, fields, arguments, ENUMS, etc..

## The alternatives

Consonance still has a conventional RESTful API, though it is no longer under development.

You can also download CSV files in fairly flexible, user-defined formats, for products and contacts.

ONIX is a popular data interchange format between supply chain partners, and may be more suitable for some purposes.

## Tools and resources

  * https://graphql.org/ for general information about GraphQL
  * [This](http://consonance-graphql-schema.s3-website-eu-west-1.amazonaws.com/) for a website documentating our GraphQL schema
  * [Insomnia](https://insomnia.rest/) is our recommended desktop client for developing, testing, and refining your queries. Other tools like [Postman](https://www.postman.com/) are popular, but currently Insomnia has the edge because of its strong support for introspection, and ability to show in-tool documentation for the API.

## Running a query

To run a query you need to send a network request with four components.

 1. The endpoint. This is the URL that you POST the query to. It is the same as the address at which you normally connect to Consonance, followed by `/graphql`. Typically it will be `https://web.consonance.app/graphql`, but check the page you normally log in at.
 2. An API key. This is your authorisation to connect and run the query, and acts as a combined user name and password, so guard it closely. You can request one from Consonance support. It's a string of thirty-two letters and numbers, like "0f9g8bhnhj6594jfjfior9d215kks3w0". In technical terms it will be used as a "bearer token" in the request.
 3. A GraphQL query which is valid for the Consonance schema. Here's a simple one, which says "give me the titles of the works": `query{works {title}}`
 4. A tool for sending the query and the api key to the end point, and receiving the result.

 For the tool we recommend Insomnia, but most programing languages will be able to send the query and get the result, as can a command line tool like `curl`.
 ```
 curl \
  -X POST \
  -H "Authorization: Bearer 0f9g8bhnhj6594jfjfior9d215kks3w0" \
  -H 'Content-Type: application/json; charset=utf-8' \
  -d '{"query":"query{works {title}}"}'\
  https://web.consonance.app/graphql
```
