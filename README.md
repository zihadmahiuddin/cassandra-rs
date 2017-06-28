[![Build Status](https://travis-ci.org/Metaswitch/cassandra-rs.svg?branch=master)](https://travis-ci.org/Metaswitch/cassandra-rs)
[![Current Version](https://img.shields.io/crates/v/cassandra-metaswitch.svg)](https://crates.io/crates/cassandra-metaswitch)
[![License](https://img.shields.io/github/license/Metaswitch/cassandra-rs.svg)](#License)

# cassandra-rs

This is a maintained Rust project that
exposes the DataStax cpp driver at https://github.com/datastax/cpp-driver/
in a somewhat-sane crate.

For the wrapper to work, you must first have installed the datastax-cpp driver.
Follow the steps in the
[cpp driver docs](https://github.com/datastax/cpp-driver/tree/master/topics#installation)
to do so. Pre-built ackages are available for most platforms.

Make sure that the driver (specifically `libcassandra_static.a` and `libcassandra.so`) are in your `/usr/local/lib64/` directory

You can use it from cargo with

```toml
    [dependencies]
    cassandra = { git = "https://github.com/metaswitch/cassandra-rs" }
```

## Documentation

You can view the API documentation by running `cargo doc` and visiting
`target/doc` in your browser.

The [Cassandra Query Language (CQL) documentation](http://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlCommandsTOC.html)
is likely to be useful.

Since this crate provides a relatively
thin wrapper around the DataStax driver, you may also find the DataStax
[documentation](http://datastax.github.io/cpp-driver/topics/) and
[API docs](http://datastax.github.io/cpp-driver/api/) useful.

## Example

Here's a straightforward example found in simple.rs:

```rust
    #[macro_use(stmt)]
    extern crate cassandra;
    use cassandra::*;
    use std::str::FromStr;


    fn main() {
        let query = stmt!("SELECT keyspace_name FROM system_schema.keyspaces;");
        let col_name = "keyspace_name";

        let contact_points = ContactPoints::from_str("127.0.0.1").unwrap();

        let mut cluster = Cluster::default();
        cluster.set_contact_points(contact_points).unwrap();
        cluster.set_load_balance_round_robin();

        match cluster.connect() {
            Ok(ref mut session) => {
                let result = session.execute(&query).wait().unwrap();
                println!("{}", result);
                for row in result.iter() {
                    let col: String = row.get_col_by_name(col_name).unwrap();
                    println!("ks name = {}", col);
                }
            }
            err => println!("{:?}", err),
        }
    }
```

There are additional examples included with the project in `tests/` and
in `src/examples`.


## License

This code is open source, licensed under the Apache License Version 2.0 as
described in [`LICENSE`](LICENSE).


## Contributing

Please see [`CONTRIBUTING.md`](CONTRIBUTING.md) for details on how to contribute 
to this project.


## Development

This crate is regularly built by Travis; to see details of the most recent builds
click on the "build" badge at the top of this page.

The unit tests assume Cassandra is running on the local host accessible on the
standard port. The easiest way to achieve this is using Docker and the standard
Cassandra image, with
```
docker pull cassandra
docker run -d --net=host --name=cassandra cassandra
```

You should run them single-threaded to avoid the dreaded
`org.apache.cassandra.exceptions.ConfigurationException: Column family ID mismatch`
error. The tests share a keyspace and tables, so if run in parallel they
interfere with each other.
```
cargo test -- --test-threads 1
```

Remember to destroy the container when you're done:
```
docker stop cassandra
docker rm cassandra
```
