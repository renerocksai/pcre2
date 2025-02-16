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


## Usage Example

```zig
const std = @import("std");

const c = @cImport({
    @cDefine("PCRE2_CODE_UNIT_WIDTH", "8");
    @cInclude("pcre2.h");
});

pub fn main() !void {
    var arg_it = std.process.args();

    _ = arg_it.next(); // skip process name

    const pattern = arg_it.next() orelse ".*";
    const subject = arg_it.next() orelse "hello, world";

    // more elaborate example: use with c.PCRE2_DOTALL:
    // const pattern = arg_it.next() orelse "(?<!\\!)\\[([^\\]]*?)\\]\\(([^)]+)\\)";
    // const subject = arg_it.next() orelse "[hello, link](https://google.com)";

    std.debug.print("pattern = '{s}', subject = '{s}'\n", .{ pattern, subject });

    var error_code: c_int = undefined;
    var error_offset: usize = undefined;

    const re_ptr = c.pcre2_compile_8(
        pattern.ptr,
        pattern.len,
        0, // or sth like c.PCRE2_DOTALL,
        &error_code,
        &error_offset,
        null,
    );
    if (re_ptr) |re| {
        defer c.pcre2_code_free_8(re);

        if (c.pcre2_match_data_create_from_pattern_8(re, null)) |match_data| {
            defer c.pcre2_match_data_free_8(match_data);

            const rc = c.pcre2_match_8(
                re,
                subject.ptr,
                subject.len,
                0, // start at offset 0 in the subject
                0, // or sth like c.PCRE2_DOTALL,
                match_data,
                null,
            );
            if (rc == c.PCRE2_ERROR_NOMATCH) {
                std.debug.print("No match!\n", .{});
            } else if (rc < 0) {
                std.debug.print("Matching error {d}!\n", .{rc});
            } else {
                const ovector = c.pcre2_get_ovector_pointer_8(match_data);
                const matching_string = subject[ovector[0]..ovector[1]];
                std.debug.print("Found match: '{s}'\n", .{matching_string});
            }
        }
    } else {
        std.debug.print("Invalid pattern {s}\n", .{pattern});
    }
}
```




