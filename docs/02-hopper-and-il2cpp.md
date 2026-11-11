# 2. Map code and inspect it (Hopper optional)

## Hopper is optional

You usually do not need Hopper. Agents can do this work with a metadata parser,
`xcrun otool`, `xcrun llvm-objdump`, and Python scripts. Use Hopper only when the
raw disassembly is too hard to follow, for example a long function with many
branches, or when you want a graphical view of callers and pseudocode.

## The workflow

1. Pair the extracted `UnityFramework` with the app's IL2CPP metadata. A metadata
   parser based on the upstream layouts recovers the `Assembly-CSharp.dll`
   type/method records.
2. Locate the assembly's code-generation module and method-pointer table in the
   native framework. Use metadata method token row IDs to resolve native addresses.
3. Select the methods that own the behavior for focused inspection.
4. Use Xcode's LLVM disassembler to inspect the surrounding control flow and
   verify any patch instructions.
5. Optional: open the extracted `UnityFramework` in Hopper as an ARM64 Mach-O
   binary, define each function as a procedure, and export its pseudocode.

Whole-framework Hopper analysis of a large Unity binary can take a very long time.
Stop it and work on selected addresses if needed. Record which procedures were
actually analyzed. One exported function does not mean the app is decompiled.

## Hopper steps you can repeat (optional)

Open the native executable, choose its correct architecture, and navigate to a
known method address. Define the function with the **P / Procedure** toolbar
control if needed, name it, and switch to the pseudocode view. Use assembly and
references alongside the pseudocode; a plausible-looking decompilation can still
misinterpret types or omit code. Save the analysis as a `.hop` database. Hopper's
[official tutorial](https://www.hopperapp.com/tutorial.html) explains the views and
procedure types.

Record each selected method's namespace, metadata token, and address. Addresses
are tied to the binary SHA-256. A file offset and a virtual address are not
generally interchangeable: convert using the containing Mach-O segment's
`fileoff` and `vmaddr`.

## How to recover the IL2CPP mapping

Use the structures in upstream Il2CppDumper's
[MetadataClass.cs](https://github.com/Perfare/Il2CppDumper/blob/master/Il2CppDumper/Il2Cpp/MetadataClass.cs)
and [Il2CppClass.cs](https://github.com/Perfare/Il2CppDumper/blob/master/Il2CppDumper/Il2Cpp/Il2CppClass.cs)
as layout references, or run a compatible metadata tool.

- The metadata header identifies the string, image, type, and method tables.
- Find the `Assembly-CSharp.dll` image, its first type index, and its type count.
- Record the type and method record strides for the metadata version.
- Locate the code-generation module and its method-pointer table.
- For a method token, take its low 24 bits as a row ID; pointer index is row ID
  minus one. Check bounds and uniqueness before trusting the mapping.
- Check that each nonzero pointer is aligned and inside an executable section.
  Null and shared generic methods need further handling; a missing pointer is not
  evidence of missing behavior.

Layouts change between metadata versions. Validate the layout for each build;
this document is not a universal metadata parser.

## Inspect assembly without Hopper

```sh
BINARY="artifacts/myapp-1.0/extracted/Payload/MyApp.app/Frameworks/UnityFramework.framework/UnityFramework"
xcrun llvm-objdump --disassemble \
  --start-address=0xSTART --stop-address=0xSTOP "$BINARY"
```

Look for matching reads and writes of the same field. For example, a time-up
check that loads a value at an object offset and compares it with 1, and an
update method that decrements the same offset, connect the countdown state to
the decision that uses it. That connection is what makes a state patch a valid
target.

A Hopper scripting lesson: marking bytes with `TYPE_PROCEDURE` did not necessarily
create a procedure object. Use Hopper's Procedure control and check that
`getProcedureAtAddress` returns a procedure before calling `decompile()`.
