# HSA Profiler May 2015 Beta Release

This beta release of the HSA Profiler is compatible with the May 2015 HSA
runtime release. This is a near-feature complete version of the profiler with
the HSA Profiling feature set nearly on par with the CodeXL OpenCL Profiler.

Information contained here is specific to sprofile version 3.1.4195

The HSA Profiler is integrated into the CodeXL GPU Profiler Backend (aka
"sprofile").  There are two new command-line switches used to tell sprofile to
profile an HSA application:  `--hsatrace` and `--hsapmc`

### Collecting an Application Trace

To collect an application trace with kernel timestamps:

   `./sprofile --hsatrace AppToProfile`

Executing sprofile with the `--hsatrace` switch will launch the specified
"AppToProfile" and allow the profiler to trace all HSA APIs called by the
application. In addition, the profiler will also gather 2 sets of timestamp
information:
* Host-side CPU timestamps for all HSA APIs called by the application
* Device-side GPU timestamps for all HSA kernels dispatched by the application

The results from the profiler will be written to an .atp file that is very
similar to the .atp file written by CodeXL's OpenCL profiler.  When profiling
an HSA application, the .atp file will have four main sections, the first three
of which are similar to the sections written by the OpenCL profiler. The CodeXL
documentation contains detailed information on the contents of the .atp file
for an OpenCL application.

Here is a brief description of the .atp file sections for the HSA profiler:
* The first section is a file header containing information about the application profiled, the system the profile was gathered on, and some of   the options used to gather the profile.
* The second section is the API trace, showing a per-thread list of all HSA APIs called by the application. For each API, the parameters passed and the values returned by the the API are shown.
* The third section is the host-side timestamps section, showing a per-thread list of all HSA APIs called by the application.  For each API, a start time and end time is shown.
* The fourth section is new for HSA.  It is the device-side kernel timestamp section, showing a list of each kernel dispatched by the application.  For each kernel, the following information is shown:
 * the name of the kernel symbol associated with the kernel dispatched (or "<UnknownKernelName>" if the profiler could not extract the symbol name.)
 * the kernel pointer value of the kernel dispatched.
 * the start timestamp of the kernel, indicating when the kernel started executing on the device. The timestamps represent nanoseconds.
 * the end timestamp of the kernel, indicating when the kernel finished executing on the device.  The timestamps represent nanoseconds.
 * the duration of the kernel (in nanoseconds).
 * the agent handle of the agent the kernel was dispatched to.

Most other sprofile switches will also work with the `--hsatrace` switch:

* You can control the name and location of the .atp file using sprofile's `--outputfile` switch.
* You can generate a subset of the Trace Summary pages using sprofile's `--tracesummary` and `--atpfile` switch.  For instance: `./sprofile --tracesummary --atpfile myapp.atp` will generate an API summary, a kernel summary and a top-ten kernel dispatch list from the data contained in the myapp.atp file. It will also produce a "Best Practices" summary file by applying a rules-based analysis of the .atp file.  Currently two rules are supported:
 * An error will be reported if any HSA API returns an error code
 * A resource leak will be reported for mismatched create/destroy calls (i.e. if the application calls hsa_queue_create without a corresponding hsa_queue_destroy call)
 * Similarly, you can generate a summary while collecting a trace using the following command line: `./sprofile --hsatrace --tracesummary ApptoProfile`
* The following switches should also work:
 * `--envvar`
 * `--envvarfile`
 * `--fullenv`
 * `--sessionname`
 * `--workingdirectory`
 * `--interval`
 * `--maxapicalls`
 * `--apifilterfile`
 * `--sym`
* The following switches are not applicable to the HSA Profiler:
 * `--nocollapse`
 * `--ret`

You can get more information on the switches mentioned above by reading the CodeXL User Guide or by executing:

  `./sprofile --hsatrace --help`
  
A version of the AMDTActivityLogger instrumentation library which is supported
by the HSA profiler is also included in this distribution. Using this API, you
can annotate your code and have the annotations appear in the timeline UI in
CodeXL. The latest version of the AMDTActivityLogger library also contains new
APIs for controlling the collection of profiling data from within a profiled
application. Documentation on the AMDTActivityLogger API can be found in the
distribution.

You can load an HSA .atp file into CodeXL 1.7 in order to see the application
timeline.  You can download the latest version of CodeXL 1.7 from the following
location:

    http://developer.amd.com/tools-and-sdks/opencl-zone/codexl/

Even though CodeXL 1.7 can load HSA .atp files, there will be some UI issues.
It is expected that a future version of CodeXL will fully support the HSA
Profiler.

## Collecting GPU Performance Counters

To collect GPU performance counters:

   `./sprofile --hsapmc AppToProfile`

Executing sprofile with the `--hsapmc` switch will launch the specified
"AppToProfile" and allow the profiler to collect GPU Performance Counter
data for each kernel dispatched by the application. The output file will
contain the following information for each kernel dispatched:
* the name of the kernel symbol associated with the kernel dispatched (or "<UnknownKernelName>" if the profiler could not extract the symbol name.)
* The following set of dispatch parameters and compiler stats for each kernel:
 * the thread id of the host thread that dispatched the kernel
 * the dimensions of the grid (in work-items)
 * the dimensions of the work-group (in work-items)
 * the size of the shared memory used by the work-group (the work-group group segment size)
 * the number of vector GPRs used by the kernel
 * the number of scalar GPRs used by the kernel
* The values of any performance counters collected for the kernel
  

Most other sprofile switches will also work with the `--hsapmc` switch:

* You can control the name and location of the .csv file using sprofile's `--outputfile` switch.
* You can specify the performance counters to collect using the `--counterlist` switch.
* A kernel occupancy file can be generated while collecting performance counters by specifying the `--occupancy` switch.
* The following switches should also work:
 * `--envvar`
 * `--envvarfile`
 * `--fullenv`
 * `--sessionname`
 * `--workingdirectory`
 * `--interval`
 * `--maxkernels`
* The following switches are either not implemented yet or are not applicable to the HSA Profiler:
 * `--kerneloutput`
 * `--kernellistfile`
 * `--singlepass`
 * `--nogputime`

The HSA Profiler can only support collecting a set of performance counters that
can be collected in a single pass. It does not support kernel replay like the
OpenCL profiler.  Thus, if you do not specify a list of performance counters to
collect, or if the set you specify can not be collected in a single pass, the
profiler will not collect all of the counters.  It will enable as many counters
as it can fit into a single pass. To collect a full set of performance counters
for an application, you will need to profile the application multiple times and
specify a different set of counters for each profile run.  Because of this
restriction, the `--singlepass` and `--nogputime` switches are always enabled
when profiling an HSA application.

## System Setup 

This assumes you are starting from a system where you can run HSA applications
outside of the profiler. The information here provides only additional steps
you need to perform to be able to profile HSA applications.

In order to profile, you will need to make sure that both the HSA runtime
libraries (libhsa-runtime64.so, libhsa-runtime-ext64.so, and libhsakmt.so) and
the HSA runtime tools library (libhsa-runtime-tools64.so) can be found and
loaded by the application you are profiling.  It is recommended that the
LD_LIBRARY_PATH environment variable contains the following two directories:
* The location of libhsa-runtime-tools64.so.1 (i.e. HSA-Profiler-AMD/lib)
* The location of libhsa-runtime64.so.1 (i.e. HSA-Runtime-AMD/lib)

**Note**: the runtime tools library (libhsa-runtime-tools64.so.1) is tightly
      coupled to the runtime library (libhsa-runtime64.so.1).  It is expected
      that if you receive an update to one of them, you will need to also get
      an update to the other.

## Sample Usage

You can profile the vector_copy sample contained in HSA-Runtime-AMD/sample
using the following steps:
 * Build the vector_copy sample using make
 * Verify that the sample executable runs
 * Execute `./sprofile --hsatrace vector_copy`
 * Execute `./sprofile --hsapmc vector_copy`

## Known Issues
* The resource tracker analysis module may report a false positive resource leak.
* Kernel occupancy information will only be written to disk if the application being profiled calls hsa_shut_down
