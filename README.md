# PLC-PID-autotuner
Portable PID autotuner using the relay method written in iec61131-3 structured text format

# What does it do
It tunes PIDs using the Relay Method[1]. It can tune them live, so you can use it to do automatic online tuning of PIDs.
It is not the most advanced method of doing so, but it will produce reasonable results for a lot of systems.
It is also very easy to understand and use.

# How do i use it
The code is in PLCOpen XML files, which should be readily importable into a large number of PLC environments. You will likely need to change how it records the current time in frequency measurement, but that is likely to be it.

# Why does this exist
1. There is a lot of interest in this kind of thing.
1. Almost all code you can find on the internet to do this for PLC's is badly broken. Seriously. If you google search and then go through the results, there is a bunch of code out there that is just truly horribly broken. It looks like it might work, but for example, it has a frequency measurement function that only counts rising edges and so gives you wrong results, or ....
1. I needed to tune a PID for a vacuum pump, and also needed an auto-tuning PID for a temperature controller for a smoker.

# What is left to do
1. Nothing, it should work and has been tested pretty extensively.
1. Some day it may be worth the time for someone to implement SIMC and other methods. The current tuning works well for my purposes (where the setpoint is not changing). If you have a system where you need good response to changing setpoint, you need more than what i'm giving you.  However, the code here should be pretty easy to use as a model for adding those tuners (computing the time delay + 63% should be a pretty easy change to FB_FREQ_MEASURE, and adding step response). If you do, send me a pull request!

# How do i contribute
Send me pull requests.

# How does it work
First some background.
*ALL* PID tuning is about mathematical modeling of functions.  In particular, they are trying to understand what the function is that describes the relationship between "thing you are driving" and "system response".
To try to make it simple and concrete, let's assume we are talking about a PID for a motor that pumps water.  The faster the motor goes, the more water that is pumped.  We are using a PID to control the motor, and the input to the PID is "how many gallons of water are being pumped per second".

Here the thing we are driving is the motor, and the system response is how much water is being pumped.  If we can understand the function that relates these two variables, we can make an optimal PID for it (assuming one can exist), where optimal means "no overshoot, no undershoot, responds exactly as needed instantly".

Various methods of modeling these relationships (first order + time delay, etc) exist.  They then calculate a model for the function, and then translate it into PID constants. Many PhD thesis have been writen on this, all of them claiming to be the best way.  In the time it took you to read this file, 5 more pid tuning methods probably popped up.

The particular way we calculate the model for the function is called the relay method.  It's actually quite simple.  Simpler function models just want a parameter called ultimate gain. This is the gain that causes the system to start oscillating.  People often resort to trying to find it by hand. 
The relay method works backwards - it forces the system to oscillate, then computes the ultimate gain from the resulting oscillation (it turns out to fairly easy to approximate well)

The exact process looks like this (for our motor above):
1. Choose a setpoint for the system that is achiveable. Let's say 50gpm.
2. Make motor go to full speed until system goes above 50gpm. Measure time at which system goes over 50gpm.
3. Make motor go to zero speed until system falls below 50gpm. Measure time at which system falls back to 50pm.
4. The difference in times is the oscillation period.

Repeat this a bunch of times and average the results. This will give you an average oscillation period/frequency.
The resulting oscillation period + amplitudes of the of process and control variables are then used to compute ultimate gain.
The ultimate gain is used to compute the pid constants.

You don't have to use full speed, we allow any variant of lower and upper amplitude (so that you can make it safe)
As mentioned above, there are other models/methods.  This method works well for systems that have simple responses and where you don't really change the setpoint.



[1] See https://www.controleng.com/articles/relay-method-automates-pid-loop-tuning/ and such.
