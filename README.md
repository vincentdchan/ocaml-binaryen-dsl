
# OCaml Binaryen DSL


This library helps you to write Webassembly text format(WAT) in OCaml source code with DSL. It helps you to write a code generator for your language to WASM.

You can generate it with [Binaryen](https://github.com/WebAssembly/binaryen).

Example:

```OCaml
open Binaryen_dsl.Dsl

let codegen_retain_object m =
  let module Dsl =
    Binaryen(struct
      let ptr_ty = i32
      let m = m
    end)
  in
  let open Dsl in
  let _ = def_function "retain_counter" ~params:[ ptr_ty ] ~ret_ty:none (fun ctx ->
  let next_count = def_local ctx i32 in
  block [
    (* next_count <- obj->strong_counter *)
    local_set next_count (I32.load ~offset:8 (Ptr.local_get 0));

    (* next_count <- next_count + 1 *)
    local_set next_count I32.((local_get next_count) + (const_i32_of_int 1));

    (* obj->strong_counter <- next_count *)
    I32.store ~offset:8 ~ptr:(Ptr.local_get 0) (I32.local_get next_count);

  ]
  )
  in
  let _ = export_function "retain_counter" "retain_counter" in ()

let () = 
  let m = module_create() in
  codegen_retain_object m;
  print_endline (emit_text m)
```

Generating WAT:
```wat
(module
 (type $i32_=>_none (func (param i32)))
 (export "retain_counter" (func $retain_counter))
 (func $retain_counter (param $0 i32)
  (local $1 i32)
  (local.set $1
   (i32.load offset=8
    (local.get $0)
   )
  )
  (local.set $1
   (i32.add
    (local.get $1)
    (i32.const 1)
   )
  )
  (i32.store offset=8
   (local.get $0)
   (local.get $1)
  )
 )
)
```

# Install

## Before

Before intalling, make sure you have install [Binaryen](https://github.com/WebAssembly/binaryen).

### Install Binaryen on macOS

```bash
brew install binaryen
```

## Install from source

Clone the repo and type:

``` bash
opam install .
```

## Install from opam

```bash
opam install binaryen_dsl
```

# Usage

## Creating a module

```OCaml
let m = module_create()
```

## Provide the pointer type and module

If you want to generate wasm32, make your pointer a `i32`,
`i64` for wasm64.

``` OCaml
let module Dsl =
  Binaryen(struct
    let ptr_ty = i32
    let m = m
  end)
in
let open Dsl in
```

When you open the `Dsl`, all the WAT sematics will be exposed.

If you want to express `i32.load`, then write `I32.load`.
Most of them are very similar.

## Emit Code

You can emit binary code to a file or just emit WAT.

```OCaml
val emit_binary: module_ -> string -> unit
val emit_text: module_ -> string
```
