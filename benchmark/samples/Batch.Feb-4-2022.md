
## “Signal to High Prio Thread” benchmark measurements

</br>

Made on February 4, 2022</br>
Configuration: standard seL4</br>
sel4ench-manifest: version adb9679d2ec3a56fbbcab27fb0639b4d4e73c3c1 of Dec 8, 2021

</br>

The work contains comparison of "Late Processing" and "Early processing" algorithms.

</br>

"Late Processing" is the current algorithm. "Early Processing" is the alternative one that doesn't collect samples
into an array, but instead, in every measurement loop it:

    - re-visits current Minimum and Maximum
    - accumulates Sample and squared Sample

