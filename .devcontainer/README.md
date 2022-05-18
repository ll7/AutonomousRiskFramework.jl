# .devcontainer README

## reopen

`F1`: Reopen and rebuild in container.

better approach:

```shell
julia
include("install.jl")
```

wrong approach:
```shell
julia
]
activate .
instantiate
<backspace>
include("install.jl")
```