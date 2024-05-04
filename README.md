
# HTTP

*HTTP client and server functionality for Julia*

| **Documentation**                                                         | **Build Status**                                                                                |
|:-------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------:|
| [![][docs-stable-img]][docs-stable-url] [![][docs-dev-img]][docs-dev-url] | [![][github-actions-ci-img]][github-actions-ci-url] [![][codecov-img]][codecov-url] |

## Quick Start Guide

1. Install Julia from the offical Julia website: https://julialang.org/downloads/
2. Add and import the HTTP Package 
   ```julia
   julia> import Pkg
   julia> Pkg.add("HTTP")
   julia> using HTTP
   ```
3. Set up and a server and client:
    - Start the server in one instance of Julia
    
    ```julia
    using HTTP
    
    function handle_request(req)
        return HTTP.Response(200, "Hello World!")
    end
    
    HTTP.serve(handle_request, "localhost", 8080)
    ```
   ~This server will listen on port 8080 and respond "Hello World" to any incoming requests!~
   
    - Start the client in another instance of Julia
    ```julia
    using HTTP
    
    url = "http://localhost:8080"
    response = HTTP.get(url)
    println("Response Status: ", response.status)
    println("Response Body: ", String(response.body))
    ```

## Installation

The package can be installed with Julia's package manager,
either by using the Pkg REPL mode (press `]` to enter):
```
pkg> add HTTP
```
or by using Pkg functions
```julia
julia> using Pkg; Pkg.add("HTTP")
```

## Project Status

The package has matured and is used in many production systems.
But as with all open-source software, please try it out and report your experience.

The package is tested against current Julia LTS (1.6), and current master on Linux, macOS, and Windows.

## Contributing and Questions

Contributions are very welcome, as are feature requests and suggestions. Please open an
[issue][issues-url] if you encounter any problems or would just like to ask a question.


## Client Examples

[`HTTP.request`](https://juliaweb.github.io/HTTP.jl/stable/index.html#HTTP.request-Tuple{String,HTTP.URIs.URI,Array{Pair{SubString{String},SubString{String}},1},Any})
sends a HTTP Request Message and returns a Response Message.

```julia
r = HTTP.request("GET", "http://httpbin.org/ip")
println(r.status)
println(String(r.body))
```

[`HTTP.open`](https://juliaweb.github.io/HTTP.jl/stable/index.html#HTTP.open)
sends a HTTP Request Message and
opens an `IO` stream from which the Response can be read.

```julia
HTTP.open(:GET, "https://tinyurl.com/bach-cello-suite-1-ogg") do http
    open(`vlc -q --play-and-exit --intf dummy -`, "w") do vlc
        write(vlc, http)
    end
end
```


## Server Examples

[`HTTP.Servers.listen`](https://juliaweb.github.io/HTTP.jl/stable/index.html#HTTP.Servers.listen):

The server will start listening on 127.0.0.1:8081 by default.

```julia
using HTTP

# start a blocking server
HTTP.listen() do http::HTTP.Stream
    @show http.message
    @show HTTP.header(http, "Content-Type")
    while !eof(http)
        println("body data: ", String(readavailable(http)))
    end
    HTTP.setstatus(http, 404)
    HTTP.setheader(http, "Foo-Header" => "bar")
    HTTP.startwrite(http)
    write(http, "response body")
    write(http, "more response body")
end
```

[`HTTP.Handlers.serve`](https://juliaweb.github.io/HTTP.jl/stable/index.html#HTTP.Handlers.serve):
```julia
using HTTP

# HTTP.listen! and HTTP.serve! are the non-blocking versions of HTTP.listen/HTTP.serve
server = HTTP.serve!() do request::HTTP.Request
   @show request
   @show request.method
   @show HTTP.header(request, "Content-Type")
   @show request.body
   try
       return HTTP.Response("Hello")
   catch e
       return HTTP.Response(400, "Error: $e")
   end
end
# HTTP.serve! returns an `HTTP.Server` object that we can close manually
close(server)
```

## WebSocket Examples

```julia
julia> using HTTP.WebSockets
julia> server = WebSockets.listen!("127.0.0.1", 8081) do ws
        for msg in ws
            send(ws, msg)
        end
    end

julia> WebSockets.open("ws://127.0.0.1:8081") do ws
           send(ws, "Hello")
           s = receive(ws)
           println(s)
       end;
Hello

julia> close(server)
```

[docs-dev-img]: https://img.shields.io/badge/docs-dev-blue.svg
[docs-dev-url]: https://JuliaWeb.github.io/HTTP.jl/dev

[docs-stable-img]: https://img.shields.io/badge/docs-stable-blue.svg
[docs-stable-url]: https://JuliaWeb.github.io/HTTP.jl/stable

[github-actions-ci-img]: https://github.com/JuliaWeb/HTTP.jl/workflows/CI/badge.svg
[github-actions-ci-url]: https://github.com/JuliaWeb/HTTP.jl/actions?query=workflow%3ACI

[codecov-img]: https://codecov.io/gh/JuliaWeb/HTTP.jl/branch/master/graph/badge.svg
[codecov-url]: https://codecov.io/gh/JuliaWeb/HTTP.jl

[issues-url]: https://github.com/JuliaWeb/HTTP.jl/issues
