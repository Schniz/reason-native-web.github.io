---
id: with-routes-package
title: Useage with Routes library
---

If you want a nice performant router you can use the [Routes](https://github.com/anuragsoni/routes) library from [anuragsoni](https://github.com/anuragsoni).

First step is to create handlers. A handler is just a function that returns a `Response.t`.

> Note: We recommend that you include request and context in all handler signatures even if they are not used. That will make it easier to handle things like databases in the future.

```ocaml
/* Just respond with "ok" on every request */
let root_handler = (_request, _context) => Http.Response.Ok.make();

/* Return a greeting with the name */
let greet_handler = (greeting, _request, _context) => {
  Http.Response.Text.make(greeting);
};

```

Then you create a route definition.

```ocaml
let routes =
  Routes.(
    Routes.Infix.(
      with_method([
        (`GET, root_handler <$ s("")),
        (`GET, greet_handler <$> s("greet") *> str),
      ])
    )
  );
```

Lastly you create a routes callback and start the server. In this case we pass in the request and context even if they are not used.

```ocaml
let routes_callback = (~request: Http.Request.t, context) => {
  Routes.match_with_method(~target=request.target, ~meth=request.meth, routes)
  |> (
    fun
    | Some(res) => res(request, context)
    | None => Http.Response.NotFound.make()
  );
};

Http.Server.start(~context=(), routes_callback) |> Lwt_main.run;
```
