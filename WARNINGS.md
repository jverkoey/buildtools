# Buildifier warnings

--------------------------------------------------------------------------------

## <a name="duplicated-name"></a>A rule with name `foo` was already found on line

### Background

Each label in Bazel has a unique name, and Bazel doesn’t allow two rules to have
the same name. With macros, this may be accepted by Bazel (if each macro
generates different rules):

```
my_first_macro(name = "foo")
my_other_macro(name = "foo")
```

Although the build may work, this code can be very confusing. It can confuse
users reading a BUILD file (if they look for the rule “foo”, they may read see
only one of the macros). It will also confuse tools that edit BUILD files.

### How to fix it

Just change the name attribute of one rule/macro.

### How to disable this warning

You can disable this warning by adding `# buildozer: disable=duplicated-name` on
the line or at the beginning of a rule.

--------------------------------------------------------------------------------

## <a name="constant-glob"></a>Glob pattern has no wildcard ('*')

[Glob function]
(https://docs.bazel.build/versions/master/be/functions.html#glob)
is used to get a list of files from the depot. The patterns (the first argument)
typically include a wildcard (* character). A pattern without a wildcard is
often useless and sometimes harmful.

To fix the warning, move the string out of the glob:

```
- glob(["*.cc", "test.cpp"])
+ glob(["*.cc"]) + ["test.cpp"]
```

**There’s one important difference**: before the change, Bazel would silently
ignore test.cpp if file is missing; after the change, Bazel will throw an error
if file is missing.

If `test.cpp` doesn’t exist, the fix becomes:

```
- glob(["*.cc", "test.cpp"])
+ glob(["*.cc"])
```

which improves maintenance and readability.

If no pattern has a wildcard, just remove the glob. It will also improve build
performance (glob can be relatively slow):

```
- glob(["test.cpp"])
+ ["test.cpp"]
```

### How to disable this warning

You can disable this warning by adding `# buildozer: disable=constant-glob` on
the line or at the beginning of a rule.

--------------------------------------------------------------------------------

## <a name="positional-args"></a>Keyword arguments should be used over positional arguments

All top level calls (except for some built-ins) should use keyword args over
positional arguments. Positional arguments can cause subtle errors if the order
is switched or if an argument is removed. Keyword args also greatly improve
readability.

```
- my_macro("foo", "bar")
+ my_macro(name = "foo", env = "bar")
```

The linter allows the following functions to be called with positional
arguments:

*   `load()`
*   `vardef()`
*   `export_files()`
*   `licenses()`
*   `print()`

### How to disable this warning

You can disable this warning by adding `# buildozer: disable=positional-args` on
the line or at the beginning of a rule.

--------------------------------------------------------------------------------

## <a name="load"></a>Loaded symbol is unused

### Background

[load]
(https://docs.bazel.build/versions/master/skylark/concepts.html#loading-an-extension)
is used to import definitions in a BUILD file. If the definition is not used in
the file, the load can be safely removed. If a symbol is loaded two times, you
will get a warning on the second occurrence.

### How to fix it

Delete the line. When load is used to import multiple symbols, you can remove
the unused symbols from the list. To fix your BUILD files automatically, try
this command:

```
buildozer 'fix unusedLoads' path/to/BUILD
```

If you want to keep the load, you can disable the warning by adding a comment
`# @unused`.

### How to disable this warning

You can disable this warning by adding `# buildozer: disable=load` on the line
or at the beginning of a rule.

--------------------------------------------------------------------------------

## <a name="unused-variable"></a>Variable is unused

This happens when a variable is set but not used in the file, e.g.

```
x = [1, 2]
```

The line can often be safely removed.

If you want to keep the variable, you can disable the warning by adding a
comment `# @unused`.

```
x = [1, 2] # @unused
```

### How to disable this warning

You can disable this warning by adding `# buildozer: disable=unused-variable` on
the line or at the beginning of a rule.

--------------------------------------------------------------------------------

## <a name="redefined-variable"></a>Variable has already been defined

### Background

In .bzl files, redefining a global variable is already forbidden. This helps
both humans and tools reason about the code. For consistency, we want to bring
this restriction also to BUILD files.

### How to fix it

Rename one of the variables.

Note that the content of lists and dictionaries can still be modified. We will
forbid reassignment, but not every side-effect.

### How to disable this warning

You can disable this warning by adding `# buildozer: disable=unused-variable` on
the line or at the beginning of a rule.

--------------------------------------------------------------------------------

## <a name="package-on-top"></a>Package declaration should be at the top of the file

Here is a typical structure of a BUILD file:

*   `load()` statements
*   `package()`
*   calls to rules, macros

Instantiating a rule and setting the package defaults later can be very
confusing, and has been a source of bugs (tools and humans sometimes believe
package applies to everything in a BUILD file). This might become an error in
the future (but it requires large-scale changes in google3).

### What can be used before package()?

The linter allows the following to be before `package()`:

*   comments
*   `load()`
*   variable declarations
*   `package_group()`
*   `licenses()`

### How to disable this warning

You can disable this warning by adding `# buildozer: disable=package-on-top` on
the line or at the beginning of a rule.

--------------------------------------------------------------------------------

## <a name="integer-division"></a>The `/` operator for integer division is deprecated

The `/` operator is deprecated in favor of `//`, please use the latter for
integer division:

`
a = b // c
d //= e
`

--------------------------------------------------------------------------------

## <a name="no-effect"></a>Expression result is not used

The statement has no effect. Consider removing it or storing its result in a
variable.

--------------------------------------------------------------------------------

## <a name="attr-cfg"></a>`cfg = "data"` for attr definitions has no effect

The [Configuration](https://docs.bazel.build/versions/master/skylark/rules.html#configurations)
`cfg = "data" is deprecated and has no effect. Consider removing it.

--------------------------------------------------------------------------------

## <a name="attr-non-empty"></a>`non_empty` attribute for attr definitions are deprecated

The `non_empty` [attribute](https://docs.bazel.build/versions/master/skylark/lib/attr.html)
for attr definitions is deprecated, please use `allow_empty` with an opposite value instead.

--------------------------------------------------------------------------------

## <a name="attr-single-file"></a>`single_file` is deprecated

The `single_file` [attribute](https://docs.bazel.build/versions/master/skylark/lib/attr.html)
is deprecated, please use `allow_single_file` instead.

--------------------------------------------------------------------------------

## <a name="ctx-actions"></a>`ctx.{action_name}` is deprecated

The following [actions](https://docs.bazel.build/versions/master/skylark/lib/actions.html)
are deprecated, please use the new API:

  * `ctx.new_file` -> `ctx.actions.declare_file`
  * `ctx.experimental_new_directory` -> `ctx.actions.declare_directory`
  * `ctx.file_action` -> `ctx.actions.write`
  * `ctx.action(command = "...")` -> `ctx.actions.run_shell`
  * `ctx.action(executable = "...")` -> `ctx.actions.run`
  * `ctx.empty_action` -> `ctx.actions.do_nothing`
  * `ctx.template_action` -> `ctx.actions.expand_template`

--------------------------------------------------------------------------------

## <a name="package-name"></a>Global variable `PACKAGE_NAME` is deprecated

The global variable `PACKAGE_NAME` is deprecated, please use
[`native.package_name()`](https://docs.bazel.build/versions/master/skylark/lib/native.html#package_name)
instead.

--------------------------------------------------------------------------------

## <a name="repository-name"></a>Global variable `REPOSITORY_NAME` is deprecated

The global variable `REPOSITORY_NAME` is deprecated, please use
[`native.repository_name()`](https://docs.bazel.build/versions/master/skylark/lib/native.html#repository_name)
instead.

--------------------------------------------------------------------------------

## <a name="load-on-top"></a>Load statements should be at the top of the file.

Load statements should be first statements (with the exception of `WORKSPACE` files),
they can follow only comments and docstrings.

--------------------------------------------------------------------------------

## <a name="filetype"></a>The `FileType` function is deprecated

The function `FileType` is [deprecated](https://docs.bazel.build/versions/master/skylark/backward-compatibility.html#filetype-is-deprecated).
Instead of using it as an argument to the [`rule` function](https://docs.bazel.build/versions/master/skylark/lib/globals.html#rule)
just use a list of strings.

--------------------------------------------------------------------------------

## <a name="output-group"></a>`ctx.attr.dep.output_group` is deprecated

The `output_group` field of a target is [deprecated](https://docs.bazel.build/versions/master/skylark/backward-compatibility.html#disable-output-group-field-on-target)
in favor of the [`OutputGroupInfo` provider](https://docs.bazel.build/versions/master/skylark/lib/OutputGroupInfo.html).

--------------------------------------------------------------------------------

## <a name="git-repository"></a>Function `git_repository` is not global anymore

Native `git_repository` and `new_git_repository` functions are [being removed](https://docs.bazel.build/versions/master/skylark/backward-compatibility.html#remove-native-git-repository).
Please use the Starklark versions instead:

    load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository", "new_git_repository")

--------------------------------------------------------------------------------

## <a name="http-archive"></a>Function `http_archive` is not global anymore

Native `http_archive` function are [being removed](https://docs.bazel.build/versions/master/skylark/backward-compatibility.html#remove-native-http-archive).
Please use the Starklark versions instead:

    load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")
