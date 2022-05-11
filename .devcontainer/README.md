# .devcontainer README

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