
### How init() function of every PD is invoked

1) tool/sel4coreplat/__main__.py adds a system invocation "TCBWriteRegisters" with "Program Counter" parameter filled in with "entry" address
picked up from ELF file containing PD's application. During initialization of the system "Monitor" executes the invocations of "TCBWriteRegisters" for every PD.

2) Application that is intended to run inside PD is compiled with "sel4cp" library. Entry point of the application's ELF points at the code
in crt0.s, which in turn eventually calls main() function containing in the library "sel4cp".

3) main() fucntion invokes init() function provided by application.




