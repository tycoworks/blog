---

kanban-plugin: board

---

## Reader and answer

- [ ] **Reader:** someone technical in capital markets (an engineer, architect, or product person) who knows what trading platforms do, but not streaming databases, view servers, or GraphQL in depth. Confirmed 4 Nov.
- [ ] **Answer:** (Chris, 12 Nov) Yes, for satellite applications: "things that don't need … super low level latency, but still need to be streaming and event driven. I think it's a great fit for that." Not for low-latency trading, because "there's so many hops". "I don't think we need that. I think we just need an easy way to do reactive streaming stuff."

## Introduction

- [ ] **p1** Situation: post 1 built live positions on Materialize and served them to a grid through Kafka and Zilla, from off-the-shelf parts.
- [ ] **p2** Complication: "going through Kafka meant defining every view up front in Materialize, whereas many applications need arbitrary queries and joins on the fly." (post 1's conclusion)
- [ ] **p3** Question: not settled. Post 1 asked "Is there a product here? Something based on GraphQL, like Hasura but for streaming databases?" Chris, 12 Nov: "I don't know if that's the right question." The answer above is to "Is GraphQL a good fit for a view server?" (his words, 9 Nov).

## Evidence

- [ ] **E1** think: GraphQL is a great fit for satellite applications, which need streaming, event-driven data but not ultra-low latency. (12 Nov)
- [ ] **E2** think: it isn't a fit for low-latency trading, because of the hops: Materialize's SQL, then rows as JSON out of `SUBSCRIBE`, then a TypeScript backend that builds the GraphQL response, then JSON again. (12 Nov; to ask: measured, or reasoning?)
- [ ] **E3** think: the need is the one Materialize gives for itself. More agents are acting than ever, and they need up-to-the-second data, or they act on a stale view of the world. GraphQL could be a nice way for agents to get data, but MCP is the standard there. (12 Nov)
- [ ] **E4** did: no demand check yet. "I haven't spoken to anyone about this, so I don't know." (12 Nov)
