---
jupyter:
  jupytext:
    formats: jl,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.5.2
  kernelspec:
    display_name: Julia 1.5.2
    language: julia
    name: julia-1.5
---

# JSXGraph.jl

[JSXGraph.jl](https://github.com/tlienart/JSXGraph.jl) package aims to help generate JSXGraph code from Julia code that can then be plugged in a statically served website (for instance one you would generate via Franklin.jl).


```julia
]st WebIO JSXGraph
```

```julia
using WebIO # WebIO v0.8.15
using JSXGraph #JSXGraph v0.1.6
```

# Define helper functions

```julia
function initscope(id="board")
    n = Node(
        :div,
        id=id,
        className = "jxgbox",
        attributes=Dict(
            :style=>"width:500px; height:500px;margin:0 auto;"
        )
    )
    w = Scope(
        imports=[
                joinpath(dirname(pathof(JSXGraph)), "libs", "jsxgraphcore.js"),
                joinpath(dirname(pathof(JSXGraph)), "libs", "jsxgraph.css"),
        ]
    )(n)
end

function to_js4ijulia(brd::JSXGraph.Board)
    # modify script generated by `str(brd)`
    script = str(brd)[begin:end-5] # remove "();"
    script *= " return board;})" # insert " return board;})" at the end of `script`
    return script
end

@WebIO.register_renderable(JSXGraph.Board) do b
    return onmount(initscope(b.name), to_js4ijulia(b))
end

```

# Display 

```julia
brd = board("board",xlim=[-15,15], ylim=[-15,15],axis=true)

a = slider("a", [[8, 7], [12, 7], [-3, 0.1, 10]])
b = slider("b", [[8, 6], [12, 6], [-1, 1, 5]])
c = slider("c", [[8, 5], [12, 5], [-10, -5, 2]])
[a,b,c] |> brd

@jsf f(x) = val(a) * x^2 + val(b) * x + val(c)

brd ++ plot(f)

brd |> WebIO.render
```
