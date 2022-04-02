
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


#### Analysis of raw data

I made checking of the raw data for all available architectures. Parameters of measurements: 10/100 (10 warm-up samples, 100 recorded samples)


    - Armv7-A

The only problem found here was the mentioned pattern. Other samples dispersed from each other in limited way, 
but no other appearant patterns were found.

    - Armv8-A

The mentioned pattern was not found here. On the contrary, every 8-th sample was few cycles less then neighbouring ones.
In addition, 2 - 3 tailing samples were considerably higher than others. Further experiments showed that the tailing increased samples
don't depend on number of measured samples.

    - Risc-V
    
    - x86_64






