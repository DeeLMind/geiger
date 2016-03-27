geiger
=====
A micro benchmark library in C++ that supports hardware performance counters.


Build & install
---------------
```bash
$ mkdir build && cd build
$ cmake -DCMAKE_INSTALL_PREFIX=/usr ..
$ make
# make install
```


Examples
--------

### Time measurement
The simplest usage of geiger is to measure the time required for a task:

```c++
  #include "geiger/benchmark.h"
  #include "geiger/printer_console.h"

  #include <vector>
  #include <cstdlib>
  
  int main()
  {
      // This is mandatory before running any benchmarks
      geiger::init();
  
      // A benchmark suite that does only time measurement
      geiger::suite<> s;
  
      s.add("rand", []()
            {
                rand();
            });
  
      s.add("vector push_back", []()
            {
                std::vector<int> v;
                v.push_back(1000);
            });
  
      // Redirection of each test result to the "console" printer
      s.set_printer<geiger::printer::console>();
  
      // Run each test during one second
      s.run();
  
      return 0;
  }
```

This code will output:

```
  Test              Time (ns)
  ---------------------------
  rand                     14
  vector push_back         47
```

By default - as in the example above - each test is running during one second. Here, the "rand" test has then been executed
tens of thousands of times, and the average execution time was 14ns. 

You can specify the duration you want as argument to *geiger::suite::run()*...

```c++
      // Run each test during one millisecond
      s.run(std::chrono::milliseconds(1));
```

... or a number of iterations:

```c++
      // Run each test exactly 100 times
      s.run(100);
```

Simply be aware that a too short time - or a too low number of iterations - can result in less accurate measurements.

When specifying a duration, before running the benchmark, geiger is performing a calibration stage where it approximates the number of iterations required to run this task during the specified time.


---

### Hardware counters
A more advanced usage of geiger is to include hardware counters. This is done by specifying a list of *papi_wrapper<_EventT...>* in the
template parameters list of *geiger::suite<>*:

```c++
      // instr_profiler reports the number of instructions, cycles and mispredicted branches
      suite<instr_profiler> s;
      
      // As before, adding some tests...

      // Run all tests and measure hardware counters
      s.run();
```

The output is now displaying the time, but also the total of each hardware events:

```
Test              Time (ns) PAPI_TOT_INS PAPI_TOT_CYC  PAPI_BR_MSP
------------------------------------------------------------------
rand                     14   3530981306   1887375567      2245340
vector push_back         46   4642911637   1889165095         9845
```
  
You can cover the events you want by defining your own PAPI wrapper. For example, if you are interested by events
around branch predictions:

```c++
      // Measuring branches taken, not taken, mispredicted and correctly predicted
      using branch_profiler = papi_wrapper<PAPI_BR_TKN, PAPI_BR_NTK, PAPI_BR_MSP, PAPI_BR_PRC>

      suite<branch_profiler> s;
```


---

### Events
### Implementation details
