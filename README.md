# Onyx Compiler Extension Example

This project demonstrates how to create a compiler extension for the Onyx programming language. It includes an example of defining and using custom macros to extend the Onyx compiler's functionality.

## Project Structure

The compiler extension is implemented in `src/extension.onyx`.
The project is organized as follows:

```
onyx-compiler-extension-example/
├── onyx-pkg.kdl             # Package configuration file for the Onyx project
├── README.md                # Documentation for the project
├── src/                     # Source files for the extension and main program
│   ├── extension.onyx       # Implementation of the compiler extension
│   └── main.onyx            # Main program demonstrating the extension's usage
```


#### `src/extension.onyx`
Contains the implementation of the compiler extension. Key features include:
- **Macro Handling**: Defines a macro named `define_func` that generates functions dynamically.
- **Message Handling**: Processes messages from the compiler, including macro expansion requests.

#### `src/main.onyx`
Demonstrates the usage of the compiler extension. Key features include:
- **Extension Declaration**: Declares the extension and its supported macros (`double`, `define_func`).
- **Macro Usage**: Uses the `define_func` macro to define multiple functions and the `double` macro for arithmetic operations.

## How It Works

### Compiler Extension (`extension.onyx`)

1. **Initialization**:
   ```onyx
    #load "core:onyx/compiler_extension"

    use onyx.compiler_extension {*}
    use core.conv
    use core {eprintf, tprintf, Result}

    main :: () {
        ext := ExtensionContext.make("My First Extension")

        // Shorthand way of handling a macro expansion request
        ext->handle_macro("define_func", handle_define_func)

        ext->start(message_handler)
    }
   ```

2. **Macro Handling**:
   - The `define_func` macro generates functions dynamically based on the provided body.
   - Example implementation:
     ```onyx
     handle_define_func :: (
         ext: &ExtensionContext,
         em: ExpansionInfo
     ) -> Result(str, ExpansionFailureReason) {
         em.body->strip_whitespace()

         code: dyn_str

         for line in em.body->split_iter("\n") {
             line->strip_whitespace()

             code->append(line)
             code->append(" :: () { println(\"Hello from ")
             code->append(line)
             code->append("\") }\n");
         }

         return .{ Ok = code }
     }
     ```

3. **Message Handling**:
   - Processes messages from the compiler, such as macro expansion requests.
   - Example:
     ```onyx
     message_handler :: (ext, msg) => {

          // Print debugging in compiler extensions requires you to print
          // to standard error, so you have to use 'eprintf'
         eprintf("Message: {p}\n", msg)

         switch msg {
             case .ExpandMacro as em {
                 if em.macro_name == "double" {
                     em.body->strip_whitespace()
                     ext->send(.{
                         Expansion = .{
                             id = em.id,
                             code = .{
                                 Ok = tprintf("2 * ({})", em.body)
                             }
                         }
                     })

                     break
                 }

                 ext->send(.{
                     Expansion = .{
                         id = em.id,
                         code = .{ Err = .NotSupported }
                     }
                 })
             }

             case _ {}
         }
     }
     ```

### Main Program (`main.onyx`)

1. **Extension Declaration**:
   ```onyx
   Extension :: #compiler_extension "./extension.onyx" {
       double, define_func
   }
   ```

2. **Macro Usage**:
   - Defines multiple functions using the `define_func` macro:
     ```onyx
     Extension.define_func!{
         foo
         bar
         qux
         dog
     }
     ```
   - Uses the `double` macro for arithmetic:
     ```onyx
     main :: () {
         dog()

         println(Extension.double!{
             12345
         })
     }
     ```

## Running the Project

To run the project, execute the `main.onyx` file:

```shell
$ onyx run src/main.onyx
```
