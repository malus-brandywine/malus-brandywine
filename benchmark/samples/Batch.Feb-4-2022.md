
## “Signal to High Prio Thread” benchmark measurements


Made on February 4, 2022, standard seL4
sel4ench-manifest: version adb9679d2ec3a56fbbcab27fb0639b4d4e73c3c1 of Dec 8, 2021


The work contains comparison of "Late Processing" and "Early processing" algorithms.


"Late Processing" is the current algorithm. "Early Processing" is the alternative one that doesn't collect samples
into an array, but instead on every measurement loop it:

    - re-visits current Minimum and Maximum
    - accumulates Sample and squared Sample

