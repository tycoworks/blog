# Social Promo

As posted on LinkedIn when the post was first published, with `tycostream-overview.pdf` as the carousel.

## LinkedIn

In my last two posts, I explored whether a streaming database could power a trading desk UI. The answer was yes — but getting data to the frontend was fiddlier than expected.

So I've started working on a new project: tycostream — an API layer that turns Materialize views into real-time GraphQL subscriptions and triggers.

Think Hasura, but for streaming databases. Or if you're from capital markets, think a view server / data server, but built on open standards (GraphQL, Postgres wire protocol, WebSockets).

It's early, but the core is ready for feedback. Have you ever hit this problem? How did you solve it? Would something like tycostream help?

Check it out here: https://lnkd.in/eETngdes

## LinkedIn Repost

Earlier this year, I started building a tool to turn Materialize views into real-time GraphQL APIs. I hadn't worked on it in a few months, but I just finished a new version that changes how it streams data.

It now handles nested joins and Hasura-style sorting and paging, so you can subscribe to related views in one request. It also now sends Materialize's updates in complete batches, making it easier to build applications that need consistent data.

Are you building real-time applications over streaming databases? If so, how are you interacting with them, and would something like this be useful? Let me know in the comments.

Try it out here: https://github.com/tycoworks/tycostream

P.S. I also reworked the architecture a little, and updated the original post with the new design: https://tycoworks.substack.com/p/tycostream-turn-materialize-views