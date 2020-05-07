## Swift 编译

Swift 会比clang 多一个代码生成阶段，即通过SILGen生成SIL，方便针对Swift进行特定优化。SIL会被传递给IR生成阶段，然后生成LLVM IR。

生成可执行文件：

```
$ swiftc Toy.swift
$ ./toy
```

生成AST，检查语法树：

```
$ swiftc -dump-ast Toy.swift
```

```
(source_file "Toy.swift"
  (import_decl range=[Toy.swift:8:1 - line:8:8] 'Foundation')
  (top_level_code_decl range=[Toy.swift:10:1 - line:10:12]
    (brace_stmt range=[Toy.swift:10:1 - line:10:12]
      (call_expr type='()' location=Toy.swift:10:1 range=[Toy.swift:10:1 - line:10:12] nothrow arg_labels=_:
        (declref_expr type='(Any..., String, String) -> ()' location=Toy.swift:10:1 range=[Toy.swift:10:1 - line:10:1] decl=Swift.(file).print(_:separator:terminator:) function_ref=single)
        (tuple_expr implicit type='(Any..., separator: String, terminator: String)' location=Toy.swift:10:6 range=[Toy.swift:10:6 - line:10:12] names='',separator,terminator
          (vararg_expansion_expr implicit type='[Any]' location=Toy.swift:10:7 range=[Toy.swift:10:7 - line:10:7]
            (array_expr implicit type='[Any]' location=Toy.swift:10:7 range=[Toy.swift:10:7 - line:10:7] initializer=**NULL**
              (erasure_expr implicit type='Any' location=Toy.swift:10:7 range=[Toy.swift:10:7 - line:10:7]
                (string_literal_expr type='String' location=Toy.swift:10:7 range=[Toy.swift:10:7 - line:10:7] encoding=utf8 value="hi!" builtin_initializer=Swift.(file).String extension.init(_builtinStringLiteral:utf8CodeUnitCount:isASCII:) initializer=**NULL**))))
          (default_argument_expr implicit type='String' location=Toy.swift:10:6 range=[Toy.swift:10:6 - line:10:6] default_args_owner=Swift.(file).print(_:separator:terminator:) param=1)
          (default_argument_expr implicit type='String' location=Toy.swift:10:6 range=[Toy.swift:10:6 - line:10:6] default_args_owner=Swift.(file).print(_:separator:terminator:) param=2))))))
```

生成LLVM IR：

```
$ swiftc -emit-ir Toy.swift
```

生成汇编：

```
$ swiftc -emit-assembly Toy.swift
```

