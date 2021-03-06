# Declarative Build System
We want to replace Build.pm by a declarative build system driven by entries in the META6.json.
From systemd-init we can learn that this will probably need a lot of directives to cover all use cases but that this is still better than each dist implementing it new in a (badly maintained) build script.

# Suggested structure in the META data
The top level "build" key is the entry point for all build-related directives. It points to a hash possibly containing the following keys:
For example:
```json
{
    "build": {
        "makefile-variables": {
            "p5helper": {"library": "p5helper"},
            "perlopts": {"run": "perl -MExtUtils::Embed -e ccopts -e ldopts"}
        }
    }
}
```

## "makefile-variables"
A hash containing names and values of variables that may be used in a Makefile.in to generate a Makefile.
The key "p5helper" will cause the string "%p5helper%" in Makefile.in to be replaced by the value given by the META file.
Values may be literals or hashes which further specify how to generate the value.

### library
library values specify the names of resources which are native libraries. They will be expanded using $*VM.plattform-library-name to match what NativeCall does.
E.g. "p5helper" will be expanded to "resources/libraries/libp5helper.so" on Linux or "resources\libraries\p5helper.dll" on Windows.
Rationale: use as a Makefile target or an output option for a compiler.

### run
Runs the given command on a shell and uses the output as value for the Makefile.in variable.
The command can be a literal or again a hash for selection by plattform specifics.
The value hash would consist of a single key and a single value which is again a hash.

Keys may be:
* "by-kernel.name"
* "by-kernel.version"
* ...
These refer to the $*KERNEL Perl 6 variable and its properties. Access to other, yet to be determined variables is expected to be useful.

The values hash contains the different cases. E.g. "windows" or "linux" for kernel.name or an empty string as fallback if no other key matches.

The plattform specifics hashes may be nested for building a selection tree.

For example:
```json
{
  "perlopts": {
    "run": {
      "by-kernel.name": {
        "windows": "perlopts.bat",
        "linux": {
          "by-kernel.version": {
            4: "perlopts-new-kernel.sh",
            2.6: "perlopts-old-kernel.sh",
            "": "perlopts-ancient-kernel.sh",
          }
        }
      }
    }
  }
}
```
