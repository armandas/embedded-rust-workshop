# Panicking

When an unrecoverable error happens in Rust, the program panics.
There are several ways that panics can be handled, but it usually ends up with your program exiting (crashing).

Let's try it. Add this before the "hello world" message:

```rust
panic!("Oh no!");
```

We should observe two things:

1. Compiler gives us "unreachable statement" warning for the code after `panic!()`.
2. When we run our program, it doesn't appear to do anything.

Let's fix point 2 by displaying the panic information. First, let's import `error` macro from the `log` crate:

```rust
use log::{info, error};
```

Then, modify the panic handler, located just after import statements, like so:

1. Give a name to the `PanicInfo` variable. In Rust, we use `_` when we have unused variables that we don't want to name.
2. Print the panic info using the `error!()` macro.

```rust
#[panic_handler]
fn panic(panic_info: &core::panic::PanicInfo) -> ! {
    error!("{panic_info}");
    loop {}
}
```

Now, we get a useful message about why our program crashed:

```
ERROR - panicked at src/bin/main.rs:32:9:
Oh no!
```

Finally, don't forget to delete the `panic!()` call before proceeding to the next section!
