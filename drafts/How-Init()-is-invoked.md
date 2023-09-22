
### How init() function of a PD's application is invoked

1) `tool/sel4coreplat/__main__.py` adds a system invocation `TCB_WriteRegisters` with `Program Counter` parameter filled in with `entry` address
picked up from an ELF file containing PD's application.

2) Application that is intended to run inside PD is compiled with `sel4cp` library. Entry point of the application's ELF points at the code
in `crt0.s`, which in turn eventually calls `main()` function contained in the library `sel4cp`.

3) `main()` invokes `init()` provided by the application.




