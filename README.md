# Tiora

Tiora is an opinionated application framework based on commands, events, and queries.

The operations that most software systems need to perform can be cleanly separated into these three concepts.

**Commands** are named after an imperative verb, sometimes but not always, on a noun. They can have arguments, but ever return a value.

Like commands, **queries** are commonly named in the form verb-noun, and they too can have arguments. However, they are always expected to return a value to the caller.

**Events** are named in the past tense of a verb, and generally refer to a noun as well. They are can happen at any time but are often the result of a command execution. They are simple DTOs describing.


Although most systems implement these concepts in various forms, the flexibility of common programming languages oftentimes makes it difficult for developers to stick to a consistent implementation, often causing undue complexity and other problems.

This framework aims to provide constructs for a consistent approach to building scalable software systems using those 3 simple abstractions.

For example, consider a User aggregate:

```typescript
class User {
  public email:string;
  constructor(args) {
    if(!args.email) throw new Error('Missing email');
    this.email = args.email;
  }
}

class RegisterUser implements ICommand {
    private user:User;
    constructor(args) {
      this.user = new User(args);
    }
    async execute(cxt:ICommandContext) {
        const existing = await cxt.query('SearchUsers', {email:this.user.email});
        if(existing.length) throw new Error('User already exists');
        // insert into database....
        // await user.save();
        cxt.log.trace('User exists, sending email');
        cxt.send('EmailWelcome', {to:this.email});
        cxt.publish(new UserRegistered('123', this.email));
        cxt.log.info('User registration completed');
    }
}

class SearchUsers implements IQuery {
    opts: QueryOptions;
    constructor(args) {
        this.opts = new QueryOptions(args);
    }
    async execute(cxt:IQueryContext) {
        //search users DB
        return [{
            email: 'hello'
        }]
    }
}

class UserRegistered implements IEvent {
    constructor(public userId, public email) {}
}

```

We have cleanly and unobtrusively defined a common operation within the bounds of a restricted domain language. This is very powerful, and opens up many opportunities.

For example, we can use our operations to build a WebSockets server:

```typescript
const endpoint = new WebsocketEndpoint('/users');
const server = new Server(endpoint);
server.registerCommand(RegisterUser);
server.registerQuery(SearchUsers);

await server.listen();
```

Why not a RESTful HTTP server?

```typescript
const endpoint = new RestEndpoint('/users');
const server = new Server(endpoint);

server.registerCommand(RegisterUser);
server.registerQuery(SearchUsers);

endpoint.route('/:id', 
await server.listen();
```

TODO: Client Side
