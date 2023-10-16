# go-hexagonal-webapp-experiment
Me experimenting with hexagonal architecture while making a web app

# Hexagonal Architecture

My interpretation, so far...

Hexagonal architecture is useful for long-lived, complex apps which merit the investment in understanding and applying its patterns. The app I am building to demonstrate it may not be either. My early impression is that it is not suitable for a legit microservice, but it is potentially useful for a monolith.

A hexagonal architecture begins with the *application* AKA the "thingy you are building."

The application has a *core* which is where its business logic should be entirely contained. Any interaction between the core and external things should be described by *ports*.

Adapters bridge the gap between the core and external things. Adapters implement interactions described by ports with the core and external things, like databases, http interfaces, caches, GUI's, etc...

The *core* should not directly depend on any adapter code. This way, one adapter implementation can be swapped for another. The application's business logic is not tied to any particular adapter technology.

Takeaway: Adapters can depend on core code, but core code should not have any dependencies on adapter code.

Adapters can be generalized as *driver* (I think of as "input") adapters which translate technology-specific requests to core API requests and *driven* (I think of as "output") adapters where the core makes a technology-agnostic request that the adapter translates into a technology-specific request. An HTTP interface, like a REST API handler, is an example of a *driver actor*. An implementation of managing persistence of domain objects using a particular database technology is an example of a *driven actor*.

## Suggested Go Source Layout

The most minimal setup would be something like

```
<package>/
            cmd/
                ...
            internal/
                core/
                    port/
                        ...
                    ...
                adapter/
                    ...
```

- The application should always have at least one entrypoint, put that in `cmd`
- The application's core is internal and should not be accessible by other applications, so we make use of Go's convention and put its core stuff in the `internal` package to prevent other code from being able to import it.
- The `internal/core` package is where all business logic should live.
    - The `internal/core/port` package is where API contracts that the adapters implement should live. It should mainly contain interfaces and maybe some errors and constants.
- The `internal/adapter` package is for code that implements ports using external code/services.

I think everything else is pretty optional. Just don't import stuff below `adapter` anywhere in `core`.

Some `core/<sub-package>` conventions I've seen:

- `core/domain/` is where you store domain models
- `core/service/` to hold most of the business logic and organize it according to groupings like a "user service" or a "permission service" or whatever you come up with.
- `core/util` for extracting code commonly shared between services

and some more `adapter/<sub-package>` conventions I've seen:

- `adapter/handler/http` for a particular HTTP user interface, like a REST API or web interface
- `adapter/repository/<db-impl>` for implementing persistence storage using the "repository" pattern. You'd add further directory like `postgres` or `mongo`
- `adapter/cache/<cache-impl>` for implementing cache functionality. You'd add further directory like `redis` or `memory`

I think the general idea is to use a top level directory to name a type of adapter and then a nested directory to contain any implementation(s) of that adapter, typically naming it after the external thing the adapter uses.

These are all just naming conventions to organize things so they're easy to find.
