# clia-redis-macros

![License](https://img.shields.io/badge/license-MIT-green.svg)
[![Cargo](https://img.shields.io/crates/v/clia-redis-macros.svg)](https://crates.io/crates/clia-redis-macros)
[![Documentation](https://docs.rs/clia-redis-macros/badge.svg)](https://docs.rs/clia-redis-macros)

A derive to store and retrieve JSON values in redis encoded using serde.

Forked from https://github.com/Bajix/derive-redis-json

## Example

Cargo.toml:

```toml
[dependencies]
clia-redis-macros = "0.2.0"
```

main.rs:

```rust
use std::sync::Arc;

use anyhow::Result;
use deadpool_redis::{redis::cmd, Pool as RedisPool};
use clia_redis_macros::RedisJsonValue;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, RedisJsonValue)]
pub struct User {
  pub id: u64,
  pub name: String,
}

pub async fn add_user(
  redis_pool: Arc<RedisPool>,
  user: User,
) -> Result<usize> {
  let mut conn = redis_pool.get().await?;
  let res: usize = cmd("SADD")
    .arg("Users")
    .arg(&user)
    .query_async(&mut conn)
    .await?;

  Ok(res)
}

pub async fn get_users(
  redis_pool: Arc<RedisPool>,
) -> Result<Vec<User>> {
  let mut conn = redis_pool.get().await?;
  let res: Vec<User> = cmd("SMEMBERS").arg("Users").query_async(&mut conn).await?;

  Ok(res)
}
```
