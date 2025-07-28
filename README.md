# ic-turso-bindings

PoC bindings for running the SQLite compatible database [Turso](https://github.com/tursodatabase/turso) on ICP.  

See the [ic-turso-demo](https://github.com/kristoferlund/ic-turso-demo) for details on how to set up Turso for the Internet Computer.

```rust
use ic_stable_structures::memory_manager::MemoryId;
use ic_turso_bindings::{params, Builder, Connection};

thread_local! {
    static MEMORY_MANAGER: RefCell<MemoryManager<DefaultMemoryImpl>> =
         RefCell::new(MemoryManager::init(DefaultMemoryImpl::default()));
}

#[ic_cdk::query]
async fn run() {
    let memory = MEMORY_MANAGER.with_borrow(|m| m.get(MemoryId::new(0)));
    let db = Builder::with_memory(memory).build().await.unwrap();
    let connection = Rc::new(db.connect().unwrap());
    
    // Create a table
    conn.execute(
        "CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, email TEXT)",
        ()
    ).await?;

    // Insert data
    conn.execute(
        "INSERT INTO users (name, email) VALUES (?1, ?2)",
        ["Alice", "alice@example.com"]
    ).await?;

    conn.execute(
        "INSERT INTO users (name, email) VALUES (?1, ?2)", 
        ["Bob", "bob@example.com"]
    ).await?;

    // Query data
    let mut rows = conn.query("SELECT * FROM users", ()).await?;
    
    while let Some(row) = rows.try_next().await? {
        let id = row.get_value(0)?;
        let name = row.get_value(1)?;
        let email = row.get_value(2)?;
        ic_cdk::println!("User: {} - {} ({})", 
            id.as_integer().unwrap_or(&0), 
            name.as_text().unwrap_or(&"".to_string()), 
            email.as_text().unwrap_or(&"".to_string())
        );
    }
}
```

