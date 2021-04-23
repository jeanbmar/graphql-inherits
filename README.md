# @inherits Directive for Apollo GraphQL

GraphQL and Apollo Server have no built-in inheritance features.

- The `extend` keyword and Apollo `@extends` directive [add fields](https://www.apollographql.com/docs/federation/federation-spec/#federation-schema-specification) to existing types and have nothing in common with JavaScript ES6 `extends`.
- GraphQL interfaces can't (and are not intended to) share fields or resolvers.

This repo provides a custom directive to achieve inheritance.

# @inherits Directive Code

```js
const { ApolloServer, gql, SchemaDirectiveVisitor } = require('apollo-server');

const typeDefs = gql`
  directive @inherits(type: String!) on OBJECT

  type Car {
    manufacturer: String
    color: String
  }
  
  type Tesla @inherits(type: "Car") {
    manufacturer: String
    papa: String
    model: String
  }
  
  type Query {
    tesla: Tesla
  }
`;

const resolvers = {
    Query: {
        tesla: () => ({ model: 'S' }),
    },
    Car: {
        manufacturer: () => 'Ford',
        color: () => 'Orange',
    },
    Tesla: {
        manufacturer: () => 'Tesla, Inc',
        papa: () => 'Elon',
    },
};

class InheritsDirective extends SchemaDirectiveVisitor {
    visitObject(type) {
        const fields = type.getFields();
        const baseType = this.schema.getTypeMap()[this.args.type];
        Object.entries(baseType.getFields()).forEach(([name, field]) => {
            if (fields[name] === undefined) {
                fields[name] = field;
            }
        });
    }
}

const schemaDirectives = {
    inherits: InheritsDirective,
};

const server = new ApolloServer({
    typeDefs,
    resolvers,
    schemaDirectives,
});

server.listen(9449).then(({ url }) => {
    console.log(`Server ready at ${url}`);
});
```

# Sample Query

Query:

```gql
query {
  tesla {
    manufacturer
    papa
    color
    model
  }
}
```

Output:

```json
{
  "data": {
    "tesla": {
      "manufacturer": "Tesla, Inc",
      "papa": "Elon",
      "color": "Orange",
      "model": "S",
    }
  }
}
```


