# Social Promo

As posted on LinkedIn when the post was first published, with `tycostream-overview.pdf` as the carousel.

## LinkedIn

In my last two posts, I explored whether a streaming database could power a trading desk UI. The answer was yes — but getting data to the frontend was fiddlier than expected.

So I've started working on a new project: tycostream — an API layer that turns Materialize views into real-time GraphQL subscriptions and triggers.

Think Hasura, but for streaming databases. Or if you're from capital markets, think a view server / data server, but built on open standards (GraphQL, Postgres wire protocol, WebSockets).

It's early, but the core is ready for feedback. Have you ever hit this problem? How did you solve it? Would something like tycostream help?

Check it out here: https://lnkd.in/eETngdes

## LinkedIn Repost

Earlier this year I started building tycostream, which turns Materialize views into real-time GraphQL APIs.

I hadn't worked on it in a few months, but I recently reworked it to add queries alongside subscriptions, as well as support for Hasura-style joins, sorting, and paging. I've also rewritten the original post with the new design and architecture.

Are you building real-time applications over streaming databases? Would something like this be useful for you? Feedback is very welcome!

Blog post: https://tycoworks.substack.com/p/tycostream-turn-materialize-views  
GitHub: https://github.com/tycoworks/tycostream
