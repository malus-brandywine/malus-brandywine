## Channels

A *channel* enables two protection domains to interact with each other using protected procedures or notifications.
Each one connects exactly two PDs; there are no multi-party channels.


When we say a channel connects two PDs it means that each PD receives its **own** channel identifier
with which the PD refers to the channel.

In practice it might look the following way: PDs **A** and **B** are connected with a channel **E**,
a channel identifier that **A** uses for **E** may have a value **37** while **B**'s channel identifier for **E** 
may have a value **42**.


Channel identifier values are local to and unique inside a PD context, so

(1) an identifier takes a unused value from [0, 63] for the next channel the PD connects to -
the system supports a maximum of 64 channels per PD,

(2) there is no way for a PD to directly refer to another PD in the system, PDs can only refer to other PDs indirectly
if there is a channel between them.

Side comment: There's no system-wide identification of channels.


### Protected procedure

A protection domain may provide a *protected procedure* (PP) which can be invoked from another protection domain.
Up to 64 words of data may be passed as arguments when calling a protected procedure.
The protected procedure return value may also be up to 64 words.

When a protection domain calls a protected procedure, the procedure executes within the context of the providing protection domain.

A protected call is only possible if the callee has strictly higher priority than the caller.
Transitive calls are possible, and as such a PD may call a *protected procedure* in another PD from a `protected` entry point.
However the overall call graph between PDs form a directed, acyclic graph.
It follows that a PD can not call itself, even indirectly.
For example, `A calls B calls C` is valid (subject to the priority constraint), while `A calls B calls A` is not valid.

When a protection domain is called, the `protected` entry point is invoked.
The control returns to the caller when the `protected` entry point returns.

The caller is blocked until the callee returns.
Protected procedures must execute in bounded time.
It is intended that future version of the platform will enforce this condition through static analysis.
In the present version the callee must trust the callee to conform.

In general, PPs are provided by services for use by clients that trust the protection domain to provide that service.

To call a PP, a PD calls `microkit_ppcall` passing the channel identifier and a *message* structure.
A *message* structure is returned from this function.

When a PD's protected procedure is invoked, the `protected` entry point is invoked with the channel identifier and message structure passed as arguments.
The `protected` entry point must return a message structure.

### Notification

A notification is a (binary) semaphore-like synchronisation mechanism.
A PD can *notify* another PD to indicate availability of data in a shared memory region if they share a channel.

To notify another PD, a PD calls `microkit_notify`, passing the channel identifier.
When a PD receives a notification, the `notified` entry point is invoked with the appropriate channel identifier passed as an argument.

Unlike protected procedures, notifications can be sent in either direction on a channel regardless of priority.

**Note:** Notifications provide a mechanism for synchronisation between PDs, however this is not a blocking operation.
If a PD notifies another PD, that PD will become scheduled to run (if it is not already), but the current PD does **not** block.
Of course, if the notified PD has a higher priority than the current PD, then the current PD will be preempted (but not blocked) by the other PD.
