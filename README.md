# Errors As Types

Comparing Rust’s type driven errors with Java exceptions
By: Ben Simmons, Rocky Slaymaker

## Recoverable Vs Unrecoverable Errors

Firstly, Rust and Java distinguish between two main categories of error, recoverable and unrecoverable errors. As the name suggests, unrecoverable errors are deemed too critical to recover from and call for the whole program to come to a stop. This is also called a “panic” in rust terminology. Once a program panics, there is no way to stop the program from quitting. 

Java does have some unrecoverable errors, but they focus on issues with the JVM, such as an `OutOfMemoryError`, rather than programming mistakes. Rust is more strict on what errors are recoverable. An example of something that will cause a panic in Rust but is allowable in Java is indexing an array out of bounds. In Java, indexing and array out of bounds results in a `ArrayIndexOutOfBoundsException`, which can be caught and recovered from. Rust's philosophy, as a low level system language, says indexing out of bounds should never be allowable. 

Recoverable errors on the other hand are not serious enough to warrant the complete stoppage of a program and can be handled. There is often a legitimate reason for a function to fail. In Java, recoverable errors fall into the category of exceptions.

## Rust Result Type
The Rust result type is a union type with two varients `Ok` and `Err` where each varient has associated values. This is very simular to the `Option<T>` type but allows for returning information in the error case versus just None.
``` rust
enum Result<T, E> {
    Ok(T), Err(E), 
}
```


## Example

One example of a recoverable error is when getting data from over the Internet, since it is common and expected to receive an HTTP error code. For example:

### Rust
```rust
fn main() {
    let response = get_response_body();
    match response {
        Ok(body) => println!("HTML response: {}", body),
        Err(err) => println!("Error: {}", err)
    }
}

fn get_response_body() -> Result<String, String> {
    let response = // some http request
    let code = response.get_http_code();
    return match code / 100 {
        2 => Ok(response.get_body()),
        4 => Err("Error on your end".to_string()),
        5 => Err("Error on the server end".to_string()),
        _ => Err("Other".to_string())
    };
}
```

In this code, we create a function that could return the HTML body from an HTTP response. If the HTTP response is within an error range, then it will return an error value, and if not, it will print out the HTML. 

Note that the `get_response_body()` function returns a `Result` type, which needs to be unwrapped in the `match` statement in the `main()` function.

The equivalent code written in Java uses exceptions and try catch statements:

### Java
```java
public static void main(String[] args) {
    try {
        System.out.println(getResponseBody());
    } catch (IOException e) {
        System.out.println("Error: " + e.getMessage());
    }
}

public static String getResponseBody() throws IOException {
    Response response = // some http request
    int code = response.getHttpCode();
    switch (code / 100) {
        case 2:
            return response.getBody();
        case 4:
            throw new IOException("Client side error");
        case 5:
            throw new IOException("Server side error");
        default:
            throw new IOException("Other error");
    }
}
```

These errors as types help because, instead of jumping between functions and possibly being thrown all the way to the main method, they work like any other return type. As a result, they are more predictable and cannot result in code unexpectedly being skipped.

Java has the notion of checked and un-checked exceptions. The compiler only forces the programer to handle checked exceptions. In contrast, Rust requires that the programmer make an explicit choice of how to handle every result. Therefore, if the program ever crashes it is guaranteed to be from an explicit decision you made instead of an assumption—the exception crashes the program in Java, but the programmer crashes the program in Rust.  In Java, simply the lack of a try-catch is the choice to crash the program (for unchecked exceptions), but Rust forces the programmer to add code with either possibility.
