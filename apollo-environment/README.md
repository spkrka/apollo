# Apollo Environment

The `apollo-environment` module lets you create a fully functional
[`Environment`](../apollo-api/src/main/java/com/spotify/apollo/Environment.java) for an
[`AppInit`](../apollo-api/src/main/java/com/spotify/apollo/AppInit.java).

The environment will contain a `RoutingEngine` and a configurable, managed
[`Client`](../apollo-api/src/main/java/com/spotify/apollo/Client.java).

The main module is `ApolloEnvironmentModule` providing an `ApolloEnvironment` instance that can be
used to initialize an `AppInit` instance, returning a `RequestHandler` to be used with any
server module: [`http-server-jetty`](../modules/http-server-jetty).

## Configuration

key | type | required | note
--- | --- | --- | ---
`apollo.backend` | string | optional | eg., `lon3`
`apollo.logIncomingRequests` | boolean | optional | default `true`


## Example

```java
public static void main(String[] args) {
  final Service service = Services.usingName("ping")
      .withModule(HttpServerModule.create()) // TODO jetty/netty?
      .withModule(ApolloEnvironmentModule.create())
      .build();

  try (Service.Instance i = service.start(args)) {
    // Create the application (possible to get instances from i.resolve())
    final Application app = new App();

    // Create Environment and call App.init(env)
    final ApolloEnvironment env = ApolloEnvironmentModule.environment(i);
    final RequestHandler handler = env.initialize(app);

    // Create servers
    final HttpServer httpServer = HttpServerModule.server(i);

    // Servers will not be bound until these calls
    httpServer.start(handler);

    i.waitForShutdown();
  } catch (InterruptedException | IOException e) {
    // handle errors
  }
}
```

For a runnable example, see [`ExampleService`]
(src/test/java/com/spotify/apollo/example/ExampleService.java)