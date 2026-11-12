# tycostream: Turn Materialize Views into Real-Time GraphQL APIs

**Subtitle:** Building an API layer for streaming databases

---

In my last two posts ([Part 1](https://tycoworks.substack.com/p/can-a-streaming-database-power-a) and [Part 2](https://tycoworks.substack.com/p/can-a-streaming-database-power-a-f49)), I explored whether streaming databases like [Materialize](https://materialize.com/) could power a trading desk UI. The answer was a cautious yes: creating the backend logic was straightforward, but getting data into the frontend was surprisingly hard. I ended up building a custom WebSocket relay for the last mile which worked, but felt too brittle and difficult to maintain in the long term.

I wanted a better way, and so I’ve started working on a new project: [tycostream](https://github.com/tycoworks/tycostream). It turns Materialize views into real-time GraphQL APIs, so you can quickly build live, type-safe apps, agents, and dashboards without needing custom infrastructure or glue code.

It would need a lot more work for production use, but the core functionality is ready for feedback. Let’s take a look.

## How It Works

The design is inspired by view servers such as [Finos Vuu](https://vuu.finos.org/) or the [Genesis Data Server](https://docs.genesis.global/docs/develop/server-capabilities/real-time-queries-data-server/), which are common components of financial markets applications. These components excel at streaming ticking prices, positions, and orders to thousands of connected UIs with fine-grained permissions and filtering. I wanted to build something similar but using open standards: GraphQL for the API and Postgres wire protocol for the database connection.

With that in mind, tycostream works a bit like [Hasura](https://hasura.io/), but for streaming databases (currently Materialize). There are two key features:

- **Subscriptions** stream typed, real-time updates to clients, with optional `where` clauses for filtering.

- **Triggers** register webhooks that fire when data meets specific conditions, for alerting, workflow automation, or integrating with external systems.

You define the views to expose, and tycostream generates GraphQL subscriptions and mutations from their schema. For example, given a `trades` table like this:

```sql
CREATE TABLE trades (
  id INT,
  instrument_id INT,
  side TEXT,
  quantity INT,
  price NUMERIC,
  executed_at TIMESTAMP
);
```

tycostream generates a GraphQL API like this:

```graphql
type Subscription {
  trades(where: tradesExpression): tradesUpdate!
}

type Mutation {
  create_trades_trigger(input: tradesTriggerInput!): Trigger!
  delete_trades_trigger(name: String!): Trigger!
}

type Query {
  trades_triggers: [Trigger!]!
  trades_trigger(name: String!): Trigger
}

type tradesUpdate {
  operation: RowOperation!
  data: trades
  fields: [String!]!
}

type trades {
  id: Int!
  instrument_id: Int
  side: side
  quantity: Int
  price: Float
  executed_at: String
}

type Trigger {
  name: String!
  webhook: String!
  fire: String!
  clear: String
}

enum RowOperation {
  INSERT
  UPDATE
  DELETE
}

enum side {
  buy
  sell
}

input tradesTriggerInput {
  name: String!
  webhook: String!
  fire: tradesExpression!
  clear: tradesExpression
}

input tradesExpression {
  id: IntComparison
  instrument_id: IntComparison
  side: sideComparison
  quantity: IntComparison
  price: FloatComparison
  executed_at: StringComparison
  _and: [tradesExpression!]
  _or: [tradesExpression!]
  _not: tradesExpression
}

input IntComparison {
  _eq: Int
  _neq: Int
  _gt: Int
  _lt: Int
  _gte: Int
  _lte: Int
  _in: [Int!]
  _nin: [Int!]
  _is_null: Boolean
}

input FloatComparison {
  _eq: Float
  _neq: Float
  _gt: Float
  _lt: Float
  _gte: Float
  _lte: Float
  _in: [Float!]
  _nin: [Float!]
  _is_null: Boolean
}

input StringComparison {
  _eq: String
  _neq: String
  _in: [String!]
  _nin: [String!]
  _is_null: Boolean
}

input sideComparison {
  _eq: side
  _neq: side
  _in: [side!]
  _nin: [side!]
  _is_null: Boolean
}
```

## Under The Hood

tycostream is built with TypeScript using [NestJS](https://nestjs.com/), [RxJS](https://rxjs.dev/), and [Apollo Server](https://www.apollographql.com/docs/apollo-server/). NestJS provides database connectivity and WebSocket support out of the box, along with modules for authentication, observability, and deployment for later down the line. RxJS handles the reactive stream processing, and Apollo Server provides the GraphQL layer with an interactive explorer for testing queries.

The project is [architected](https://github.com/tycoworks/tycostream/blob/main/docs/architecture.md) into three core modules:

- A `database` module that manages Materialize connections and parses the data stream

- A `view` module that maintains an in-memory cache and handles client-side filtering

- An `api` module that generates the GraphQL schema from view definitions

tycostream lazily instantiates a cache and Materialize connection for each view exposed over the API. When the first request for a given view comes in, tycostream opens a new connection, caches incoming updates, and streams them to the client. Additional requests for the same view are first served a snapshot from the cache, then connected to a live stream of inserts, updates, and deletes. When requests have `where` clauses, tycostream compiles them into JavaScript functions to filter each row as it streams through.

![Architecture](architecture.png)

tycostream architecture

## Integration Testing

The project includes integration test infrastructure built on [Testcontainers](https://testcontainers.com/), [Apollo Client](https://www.apollographql.com/docs/react/), and [Express](https://expressjs.com/). You define a test scenario as:

1.  A sequence of database operations for tycostream to process

2.  The number of clients to spin up and how they should interact with tycostream’s subscriptions and triggers

3.  The expected state each client should see at the end

For example, a test with one client subscribed to positions and a loss alert trigger looks like this:

```typescript
const testEnv = await TestEnvironment.create({
  appPort: 4001,
  schemaPath: 'test/schema.yaml',
});

const client = testEnv.createClient('positions-client');

await client.subscribe('positions', {
  query: `subscription { positions { operation data { id quantity } } }`,
  expectedState: new Map([[1, { id: 1, quantity: 100 }]]),
});

await client.trigger('loss-alert', {
  query: `mutation($url: String!) { create_trigger(name: "loss", view: "positions", ...) }`,
  expectedEvents: [{ event_type: 'FIRE', data: { id: 1 } }],
});

await testEnv.waitForCompletion();
```

The test passes when all clients converge to their expected state, and fails if it times out or the liveness check detects that data has stopped flowing.

## Why This Matters

In Part 1, I asked whether we’d see business logic pulled out of applications and centralized in streaming databases. Turns out there’s already a name for this pattern: the [streaming data mesh](https://www.oreilly.com/library/view/streaming-data-mesh/9781098130718/). The idea is that individual teams can publish well-defined, versioned “data products” that anyone can use, thereby creating internal APIs for live data.

I believe streaming databases like Materialize are the unlock for this pattern. They let developers join and transform live data using standard SQL, making it much easier to build scalable, real-time applications. And as demand for streaming databases grows, I expect we’ll need an ecosystem of tools around them — API layers for access, entitlements for security, and ways to trigger logic on certain conditions. This is where tycostream fits in.

Imagine vibe coding a positions dashboard in [Retool](https://retool.com/). Under the hood, it’s a simple case of connecting workflows and UI elements to live data products via GraphQL, making streaming applications as easy to build as CRUD.

## Try It Out

Want to see tycostream in action? There’s a [positions dashboard demo](https://github.com/tycoworks/tycostream?tab=readme-ov-file#running-the-demo) in the repo you can spin up locally with a single command. It uses a lightweight simulator to generate trades and market data, a Materialize emulator instance to calculate real-time positions as materialized views, and tycostream subscriptions to stream everything to the frontend. There’s also an alerts panel showing when realized losses exceed a threshold, implemented via tycostream triggers.

![Demo](video.mov)

If you’re building real-time applications with SQL and GraphQL, I’d love to hear from you. What problems are you solving? What would make this useful for your use case? Drop me a line here or at <chris@tycoworks.com>.

Thanks for reading tycoworks! Subscribe for free to receive new posts and support my work.
