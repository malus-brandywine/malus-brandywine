
#### Task

In benchmark suite (sel4bench), benchmark "Signal to High Prio Thread" produces repeating pattern where every 8-th measurement
is considerably higher than others and makes standard deviation 150-160 ccycles for MCS kernel and 170-190 ccycles for traditional one.
The pattern was spotted on Armv7-A platform (Sabre).
Initial guess was that the issue related to eviction of a line cache. (Thesis Paper by Shade Kadish)

The task was to implement the new methodology which avoids writing a long array of raw data sequentially into memory.
Instead, while sampling processes goes on the four parameters are to be calculated:
   - sum of samples,
   - sum of squared samples,
   - min,
   - max

It would allow to calculate variance, standard deviation and mean post-process and to drop raw data.

