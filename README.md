# Errors As Types

#### Errors as Unexpected Exceptions or Expected Results in Java and Rust  
By: Ben Simmons, Rocky Slaymaker

## Recoverable Vs Unrecoverable Errors

Rust and Java both distinguish between two main categories of error: recoverable and unrecoverable errors. Unrecoverable errors force the whole program to stop. Rust calls this a "panic." Once a program panics, there is no way to prevent it from quitting. Recoverable errors, notated in Java as "exceptions," can be caught using a try-catch block if they're not serious enough to force a crash.

Java's unrecoverable errors focus on issues with the runtime, like `OutOfMemoryError`, instead of programming mistakes, while Rust is more strict on what errors are recoverable. For example, in Java, indexing an array out of bounds results in a recoverable `ArrayIndexOutOfBoundsException`. Rust's philosophy, as a low level system language, says indexing out of bounds should never be allowable. 

Java's concept of recoverable exceptions exceptions has some notable problems: 
- They break "atomicity," making malformed objects possible. For example, if you're running a method that changes two variables that are always meant to go together, but an exception happens in between the changes, the object could end up in an unexpected state if the exception is caught.
- They create an implicit control flow, making it harder to follow the call stack because exceptions can suddenly "jump" up the stack to wherever they're caught.
- Unchecked exceptions are completely invisible, both to the user and the compiler, until they happen at runtime.
- They allow for "silent failures," in which a try-catch block makes an otherwise dangerous error invisible. This allows the program to continue with bad data, which is obviously problematic.

Despite those downsides, exceptions are also useful:
- They automatically "bubble up" the call stack without needing to manually return them. 
- They are also useful for extremely unusual, unexpected scenarios, so you don't have to add a bunch of return types to every function just in case.
- You can handle multiple exception types at once by just catching their superclass.

To fix these issues, Rust uses a special return type called a Result, which is returned just like anything else. Since they are returned just like any other value, they do not cause any of the problems of Java's exceptions. They don't break atomicity, jump up the call stack, or allow for silent failures, and they require the programmer to make an explicit decision to handle them or panic.

## Rust Result Type
The Rust Result type is a union type with two varients `Ok` and `Err` where each variant has associated values. This is very simular to the `Option<T>` type but allows for returning information in the error case versus just None.
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
        4 => Err("Client side error".to_string()),
        5 => Err("Server side error".to_string()),
        _ => Err("Other error".to_string())
    };
}
```

In this code, we create a function that could return the HTML body from an HTTP response. If the HTTP response is within an error range, then it will return an error value, and if not, it will print out the HTML. 

Note that the `get_response_body()` function returns a `Result` type, which needs to be unwrapped in the `match` statement in the `main()` function.

The equivalent code written in Java uses exceptions and try-catch statements:

### Java
```java
public static void main(String[] args) {
    try {
        System.out.println("HTML response: " + getResponseBody());
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

## Conclusion

The names serve as a pretty good summary of the differences. Java's exceptions are meant to be unusual, an exception to the way the code is normally supposed to run. Rust's results are intended to just signify all the possibilities that a function might return, including the errors.
