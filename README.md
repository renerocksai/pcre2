# PCRE2 for zig

PCRE2 10.45 with build.zig & build.zig.zon

## Usage

Update your `build.zig.zon`:

```shell
zig fetch --save https://github.com/renerocksai/pcre2/archive/refs/tags/pcre2-10.45.tar.gz
```

Then, in your `build.zig`, link the library as dependency (UTF-8 example):

```zig
    const pcre2_dep = b.dependency("pcre2", .{
        .target = target,
        .optimize = optimize,
        // specify code unit with
        // possible values are:
        // - .@"8"
        // - .@"16"
        // - .@"32"
        .@"code-unit-width" = .@"8",
    });

    // we assume, your compile step is `exe`
    // make sure you link against the lib with the correct code unit width
    exe.linkLibrary(pcre2_dep.artifact("pcre2-8")); // for unicode 8

```

In your code, you import the headers like this:

```zig
const c = @cImport({
    @cDefine("PCRE2_CODE_UNIT_WIDTH", "8");
    @cInclude("pcre2.h");
});
```

Please make sure to use the code unit width specification consistently.






