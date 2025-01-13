A simple per-field graphql ratelimiter library. Meant to be used as middleware.

WIP: Support for fragmentation.

Supports custom async storage backends like redis. Check crate level documentation for more.

## Usage

```rs
use graphql_ratelimit::{Ratelimit, RatelimitResult};
use std::time::Duration;

let limiter = Ratelimit::default()
    .query(|m| {
        m.field("me", |f| {
            f.rate(5.0, Duration::from_secs(10))
                .field("posts", |q| q.cost(3.0))
        })
    })
    .mutation(|m| {
        m.field("login", |f| f.rate(4.0, Duration::from_secs(40)))
            .field("createUser", |f| {
                f.rate(3.0, Duration::from_secs(30)).cost(2.0)
            })
    });

match limiter.execute(r#"mutation { login(email: "testmail@example.com", password: "Dummy password") }"#,
    "127.0.0.1".into()).unwrap() {
    RatelimitResult::Pass(tokens) => println!("Successful. {tokens} tokens left."),
    RatelimitResult::Block(duration) => println!("Wait {duration:?} before making next
    request."),
};

limiter.cleanup();
```
