# Hamming Weight Measurement

ref: https://chipwhisperer.readthedocs.io/en/latest/tutorials/pa_dpa_1-openadc-cwlitearm.html#tutorial-pa-dpa-1-openadc-cwlitearm

* assumed relationship between power on the data lines and measured power consumption
* prove:

### Capturing Power Traces
* use the AES algorithm (it doesn’t matter what we use)
* using a loop to capture multiple traces, as well as numpy to store them.

```
setup:
=======
%run "Helper_Scripts/Setup_Generic.ipynb"
fw_path = "../hardware/victims/firmware/simpleserial-aes/simpleserial-aes-{}.hex".format(PLATFORM)
cw.program_target(scope, prog, fw_path)

```

```
capture loop
============
#Capture Traces
from tqdm import tnrange
import numpy as np
import time

ktp = cw.ktp.Basic()

traces = []
N = 1000  # Number of traces

for i in tnrange(N, desc='Capturing traces'):
    key, text = ktp.next()  # manual creation of a key, text pair can be substituted here

    trace = cw.capture_trace(scope, target, text, key)

    if trace is None:
        continue
    traces.append(trace)

#Convert traces to numpy arrays
trace_array = np.asarray([trace.wave for trace in traces])  # if you prefer to work with numpy array for number crunching
textin_array = np.asarray([trace.textin for trace in traces])
known_keys = np.asarray([trace.key for trace in traces])  # for fixed key, these keys are all the same

```


```
plot them using Bokeh
=====================

from bokeh.plotting import figure, show
from bokeh.io import output_notebook

output_notebook()
p = figure()

xrange = range(len(traces[0].wave))
p.line(xrange, traces[2].wave, line_color="red")
show(p)

```


```
# cleanup the connection to the target and scope
scope.dis()
target.dis()
```


### Trace Analysis

```
numtraces = np.shape(trace_array)[0] #total number of traces
numpoints = np.shape(trace_array)[1] #samples per trace

```

[see about hammingWeight](hammingWeight.md)

Plotting HW(hammingWeight)

* plot each of the different “classes” in a different color
* we should see if there is some location that has relatively obvious difference in Hamming weight.
* get that easily using the HW array and intermediate() function 
* let’s pretend we know already what a “good” point is ( figure out what this point should be by using the CPA attack )

```
from bokeh.plotting import figure, show
from bokeh.io import output_notebook
from bokeh.palettes import brewer

output_notebook()
p = figure()

#Must run S-Box() script first to define the HW[] array and intermediate() function

#Must adjust these points for different compilers/targets
if PLATFORM == "CWLITEARM" or PLATFORM == "CW308_STM32F3":
    plot_start = 930
    plot_end = 970
elif PLATFORM == "CWLITEXMEGA" or PLATFORM == "CW303":
    plot_start = 1370
    plot_end = 1410
elif PLATFORM == "CWNANO":
    plot_start = 590
    plot_end = 620


xrange = range(len(traces[0].wave))[plot_start:plot_end]
bnum = 0

color_mapper = (brewer['Reds'][9])
print(color_mapper)

for trace in traces:
    hw_of_byte = HW[intermediate(trace.textin[bnum], trace.key[bnum])]
    p.line(xrange, trace.wave[plot_start:plot_end], line_color=color_mapper[hw_of_byte])

show(p)
```

### Finding Average at Locations

* prove that there are differences
* plot them to see how “linear” they are 
* find and and print the averages:

```
import numpy as np

#For STM32F3 build - this point seemed to work OK. May need to modify for different targets/compliers.
if PLATFORM == "CWLITEARM" or PLATFORM == "CW308_STM32F3":
    avg_point = 953
elif PLATFORM == "CWLITEXMEGA" or PLATFORM == "CW303":
    avg_point = 1394
elif PLATFORM == "CWNANO":
    avg_point = 603



hw_list = [ [], [], [], [], [], [], [], [], []]
for trace in traces:
    hw_of_byte = HW[intermediate(trace.textin[bnum], trace.key[bnum])]
    hw_list[hw_of_byte].append(trace.wave[avg_point])

hw_mean_list = [np.mean(hw_list[i]) for i in range(0, 9)]
print(hw_list[8])

for hw in range(1, 9):
    print("HW " + str(hw) + ": " + str(hw_mean_list[hw]))

```

 plot of this to see it visually

```

from bokeh.plotting import figure, show
from bokeh.io import output_notebook

output_notebook()
p = figure(title="HW vs Voltage Measurement")
p.line(range(1, 9), hw_mean_list[1:9], line_color="red")
p.xaxis.axis_label = "Hamming Weight of Intermediate Value"
p.yaxis.axis_label = "Average Value of Measurement"
show(p)
```

## results:

*  linear plot as a result



