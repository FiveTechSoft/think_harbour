# Chapter 12: Advanced Topics

In this chapter, we will explore some of Harbour's most powerful and advanced capabilities. These topics will allow you to create more complex, robust and efficient applications, from multithreaded services to dynamic web applications and integration with native code.

## 1. Multithreading Programming

Multithreading programming allows an application to execute multiple tasks concurrently, improving performance and responsiveness, especially on systems with multiple processor cores. Harbour offers a complete set of functions for thread management and synchronization.

### Creating a Thread

The heart of multithreading in Harbour is the `hb_threadStart()` function. This function starts a new execution thread that will run in parallel to the main thread.

**Syntax:**
`hb_threadStart( <@Function()|cFunctionName>, [ <Parameter1> ], [ ... ] ) --> hThread`

- `<@Function()|cFunctionName>`: A reference to a Harbour function or the name of a C function that will execute in the new thread.
- `<ParameterN>`: Optional parameters that will be passed to the thread function.
- `hThread`: The function returns a handle for the new thread, which can be used to manage it.

**Basic Example:**

```harbour
PROCEDURE Main()
   LOCAL hThread1, hThread2

   ? "Starting threads..."

   // Start a thread that executes the Worker() function
   hThread1 := hb_threadStart( @Worker(), "Thread A", 5 )

   // Start another thread
   hThread2 := hb_threadStart( @Worker(), "Thread B", 3 )

   ? "Threads started. Waiting for them to finish."

   // Wait for both threads to finish their execution
   hb_threadJoin( hThread1 )
   hb_threadJoin( hThread2 )

   ? "All threads have finished."
   RETURN

STATIC PROCEDURE Worker( cName, nRepetitions )
   LOCAL i

   FOR i := 1 TO nRepetitions
      // Print the thread name and iteration number
      // hb_outStd() is safe to use in threads (thread-safe)
      hb_outStd( cName + ": working... " + LTrim( Str( i ) ) + hb_eol() )
      // Simulate work
      hb_idleSleep( 0.5 )
   NEXT

   hb_outStd( cName + ": finished." + hb_eol() )
   RETURN
```

### Thread Synchronization: Mutex

When multiple threads access a shared resource (like a global variable or a file), it's crucial to avoid race conditions. "Mutex" (mutual exclusion) are the most common mechanism to ensure that only one thread at a time can access a critical section of code.

- `hb_mutexCreate()`: Creates a new mutex.
- `hb_mutexLock( hMutex )`: Locks the mutex. If it's already locked by another thread, this thread will wait.
- `hb_mutexUnlock( hMutex )`: Unlocks the mutex, allowing other threads to acquire it.
- `hb_mutexDestroy( hMutex )`: Frees the mutex resources.

**Example with Mutex:**

```harbour
// Global shared variable
STATIC nCounter := 0
// Mutex to protect the counter
STATIC hMutex

PROCEDURE Main()
   LOCAL aThreads := {}
   LOCAL i

   // Create the mutex before starting threads
   hMutex := hb_mutexCreate()

   // Start 10 threads that will increment the counter
   FOR i := 1 TO 10
      AAdd( aThreads, hb_threadStart( @Incrementer() ) )
   NEXT

   // Wait for all to finish
   FOR EACH hThread IN aThreads
      hb_threadJoin( hThread )
   NEXT

   // Destroy the mutex
   hb_mutexDestroy( hMutex )

   ? "Final counter value:", nCounter // Should be 10000
   RETURN

STATIC PROCEDURE Incrementer()
   LOCAL i

   FOR i := 1 TO 1000
      // Critical section: only one thread can enter at a time
      hb_mutexLock( hMutex )
      nCounter++
      hb_mutexUnlock( hMutex )
   NEXT
   RETURN
```

## 2. Web Application Development with hbhttpd

Harbour includes an integrated web server, `hbhttpd`, that allows developing web applications and RESTful services directly in Harbour. It is found in the `contrib/hbhttpd` directory.

### Basic Structure of a Web Server

To create a web server, you need:
1.  Start the server with `hb_httpServer()`.
2.  Provide a main function that will process each incoming HTTP request.

**"Hello World" Web Example:**

```harbour
#include "hbhttpd.ch"

PROCEDURE Main()
   // Start the server on port 8080
   // The HttpHandler() function will process each request
   hb_httpServer( 8080, @HttpHandler() )
   RETURN

// Function that handles HTTP requests
FUNCTION HttpHandler( hConnection )
   LOCAL cMethod := hb_httpGetMethod( hConnection )
   LOCAL cPath   := hb_httpGetPath( hConnection )
   LOCAL cBody

   // Send response header (HTTP 200 OK, HTML content type)
   hb_httpSendHeader( hConnection, 200, "Content-Type", "text/html; charset=utf-8" )

   // Build the HTML response body
   cBody := "<html><head><title>Harbour Server</title></head>"
   cBody += "<body>"
   cBody += "<h1>Hello from hbhttpd!</h1>"
   cBody += "<p>Request method: " + cMethod + "</p>"
   cBody += "<p>Requested path: " + cPath + "</p>"
   cBody += "</body></html>"

   // Send the response body to the client
   hb_httpSendResponse( hConnection, cBody )

   // Return .T. to keep connection if it's keep-alive
   RETURN .T.
```

To compile, you must link the `hbhttpd` library. For example, with `hbmk2`:
`hbmk2 my_server.prg hbhttpd.hbc`

### Processing Form Data and JSON

`hbhttpd` facilitates access to data sent by the client, such as GET parameters or POST data.

- `hb_httpGetParam( hConnection, cParamName, [ xDefault ] )`: Gets a parameter from the URL (GET) or body (POST `application/x-www-form-urlencoded`).
- `hb_httpGetBody( hConnection )`: Gets the raw request body, useful for processing JSON or XML.

**REST API Example (JSON):**

```harbour
#include "hbhttpd.ch"
#include "hbjson.ch"

PROCEDURE Main()
   hb_httpServer( 8080, @ApiHandler() )
   RETURN

FUNCTION ApiHandler( hConnection )
   LOCAL cPath := hb_httpGetPath( hConnection )
   LOCAL cJson

   hb_httpSendHeader( hConnection, 200, "Content-Type", "application/json" )

   IF cPath == "/api/user/1"
      // Create a Harbour object and convert it to JSON
      cJson := hb_jsonEncode( { "id" => 1, "name" => "John Doe", "active" => .T. } )
      hb_httpSendResponse( hConnection, cJson )
   ELSE
      // Return a 404 error
      hb_httpSendHeader( hConnection, 404, "Content-Type", "application/json" )
      hb_httpSendResponse( hConnection, hb_jsonEncode( { "error" => "Not found" } ) )
   ENDIF

   RETURN .T.
```

## 3. Using Harbour's Garbage Collector (GC)

Harbour manages memory automatically through a Garbage Collector (GC). The GC takes care of freeing memory from "items" (strings, arrays, objects, etc.) that are no longer in use.

### How does it work?

Harbour's GC uses a "mark-and-sweep" algorithm. Periodically, or when more memory is needed, the GC performs the following steps:
1.  **Mark Phase:** Starts from the roots (local, static, public variables) and traverses all objects that can be accessed. Each reachable object is "marked" as in use.
2.  **Sweep Phase:** The GC sweeps all memory, and any object that was not marked in the previous phase is considered "garbage" and is freed.

This means that, in general, you don't need to worry about freeing memory manually.

### Relevant GC Functions

Although the GC is automatic, sometimes it's useful to interact with it.

- `hb_gcCollect()`: Forces the execution of a complete garbage collection cycle. It's rarely necessary, but can be useful in debugging situations or before operations that consume a lot of memory.
- `hb_gcInfo( <nInfoType> )`: Provides information about the GC state and memory usage. For example, `hb_gcInfo( HB_GC_INFO_MEMORY_USED )` returns the amount of memory currently in use by Harbour items.

### Considerations about Circular References

One of the advantages of Harbour's GC is that it can handle **circular references**. For example:

```harbour
oA := MyObject():New()
oB := MyObject():New()

oA:oRef := oB // oA references oB
oB:oRef := oA // oB references oA

// At this point, oA and oB reference each other.
// If there are no other references to them, they become unreachable.

oA := NIL
oB := NIL

// Harbour's GC will detect that this cycle is no longer accessible
// from any root and will free the memory of both objects.
```

In older systems based on reference counting, this scenario would create a memory leak. Harbour's GC solves it without problems.

## 4. Integration of C Code and External Libraries

One of Harbour's most powerful features is its ability to extend with code written in C/C++. This allows you to:
-   Create high-performance functions.
-   Access operating system APIs.
-   Integrate third-party libraries.

### Writing a C Function for Harbour

For a C function to be visible from Harbour, you must use Harbour's extension API.

1.  **Include `hbapi.h`**: This header contains all necessary definitions.
2.  **Use the `HB_FUNC( function_name )` macro**: This is how you declare a function that will be called from Harbour. The `function_name` will be what you use in your `.prg` code.
3.  **Interact with parameters**: Use `hb_pcount()` to get the number of parameters and `hb_param(n, <type_check>)` to access them.
4.  **Return a value**: Use `hb_ret*()` functions (e.g. `hb_retc()` for a string, `hb_retni()` for an integer) to return a value to Harbour.

**Example: A C function that adds two numbers**

`my_functions.c`:
```c
#include "hbapi.h"
#include "hbapierr.h"
#include "hbvm.h"

// Define a function called SUM_C
HB_FUNC( SUM_C )
{
   // Check that two numeric parameters were passed
   if( hb_pcount() == 2 && HB_ISNUM( 1 ) && HB_ISNUM( 2 ) )
   {
      // Get parameters as doubles
      double dNum1 = hb_parnd( 1 );
      double dNum2 = hb_parnd( 2 );

      // Return the sum
      hb_retnd( dNum1 + dNum2 );
   }
   else
   {
      // Return NIL if parameters are incorrect
      hb_ret();
   }
}
```

### Compiling and Using the Function

1.  **Compile your C file** as an object (`.o` or `.obj`).
2.  **Link the object** with your Harbour application.

With `hbmk2`, this process is very simple. Just add the `.c` file to the command line:

`hbmk2 my_app.prg my_functions.c`

`hbmk2` will take care of compiling the C and linking it correctly.

**Using the function in Harbour:**

`my_app.prg`:
```harbour
PROCEDURE Main()
   LOCAL nResult

   // Call the C function as if it were a Harbour function
   nResult := Sum_C( 10, 32 )

   ? "The result of the C sum is:", nResult // 42

   // Example of incorrect call
   ? "Incorrect call:", Sum_C( "hello" ) // Will return NIL
   RETURN
```

This integration capability opens a world of possibilities, allowing combining Harbour's high productivity with the performance and low-level access of the C language.