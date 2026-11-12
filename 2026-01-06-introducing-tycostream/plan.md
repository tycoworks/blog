---

kanban-plugin: board

---

## Reader and answer

- [ ] **Reader:** someone technical in capital markets (an engineer, architect, or product person) who knows what trading platforms do, but not streaming databases, view servers, or GraphQL in depth. Confirmed 4 Nov and 12 Nov.
- [ ] **Answer:** (Chris, 12 Nov) Yes, for satellite applications: "things that don't need … super low level latency, but still need to be streaming and event driven. I think it's a great fit for that." Not for low-latency trading, because "there's so many hops". "I don't think we need that. I think we just need an easy way to do reactive streaming stuff."

## Introduction

- [ ] **p1** Situation: post 1 built live positions on Materialize and served them to a grid through Kafka and Zilla, from off-the-shelf parts.
- [ ] **p2** Complication: "going through Kafka meant defining every view up front in Materialize, whereas many applications need arbitrary queries and joins on the fly." (post 1's conclusion)
- [ ] **p3** Question: "Is GraphQL a good fit for a view server?" (his words, 9 Nov). Leaning yes on 12 Nov: "Maybe that's a good question." His caveat: "a view server implies visualizing things… That's just one use case of this." Post 1's "Is there a product here?" is about demand, which is untested, so it stays open in the conclusion.

## Evidence

- [ ] **E1** think: GraphQL is a great fit for satellite applications, which need streaming, event-driven data but not ultra-low latency. (12 Nov)
- [ ] **E2** think: it isn't a fit for low-latency trading, because of the hops: Materialize's SQL, then rows as JSON out of `SUBSCRIBE`, then a TypeScript backend that builds the GraphQL response, then JSON again. (12 Nov; to ask: measured, or reasoning?)
- [ ] **E3** think: the need is the one Materialize gives for itself. More agents are acting than ever, and they need up-to-the-second data, or they act on a stale view of the world. GraphQL could be a nice way for agents to get data, but MCP is the standard there. (12 Nov)
- [ ] **E4** did: no demand check yet. "I haven't spoken to anyone about this, so I don't know." (12 Nov)
- [ ] **E5** think: it's the era of incremental view maintenance. "There's more data than ever before. At least more live than ever before. And just polling and spamming queries isn't going to help." He points to incremental view maintenance engines, the rise of context engines, and tools like CocoIndex. (12 Nov)
- [ ] **E6** think: on demand, "I think eventually. It might be very early still." (12 Nov)
- [ ] **E7** saw: research from 28 Oct 2026 found agents mostly pull today. They query live views over MCP, and MCP's own event push is still a draft. In finance, the agents in production are woken by work arriving in a queue, such as a failed payment at BNY or an AML alert at Nasdaq Verafin, and none found publicly reacts to live market ticks. Deephaven, KX and LSEG sell agents access to live data over MCP.
