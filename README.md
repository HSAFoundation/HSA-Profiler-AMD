HSA Profiler APITrace/Timestamps March 2015 Beta release

This initial release of the HSA Profiler is compatible with the Obsidian-based
March 2015 HSA runtime release. Currently, only API Trace/Timestamp mode is
supported in the HSA Profiler.  Future releases will add Performance Counter
collection for HSA applications.

Information contained here is sepcific to sprofile version 3.1.3966

The HSA Profiler is integrated into the CodeXL GPU Profiler Backend (aka
"sprofile").  There is a new command-line switch used to tell sprofile to
collect an application trace for an HSA application:

   ./sprofile --hsatrace AppToProfile

The above command will execute the specified "AppToProfile" and allow the
profiler to trace all HSA APIs called by the application. In addition, the
profiler will also gather 2 sets of timestamp information:
  1) Host-side CPU timestamps for all HSA APIs called by the application
  2) Device-side GPU timestamps for all HSA kernels dispatched by the application

The results from the profiler will be wriiten to an .atp file that is very
similar to the .atp file written by CodeXL's OpenCL profiler.  When profiling
an HSA application, the .atp file will have four main sections, the first three
of which are similar to the sections written by the OpenCL profiler. The CodeXL
documentation contains detailed information on the contents of the .atp file for
an OpenCL application.

Here is a brief description of the .atp file sections for the HSA profiler:
1) The first section is a file header containing information about the application
   profiled, the system the profile was gathered on, and some of the options used
   the gather the profile.
2) The second section is the API trace, showing a per-thread list of all HSA APIs
   called by the application. For each API, the parameters passed and the values
   returned by the the API are shown.
3) The third section is the host-side timestamps section, showing a per-thread
   list of all HSA APIs called by the application.  For each API, a start time
   and end time is shown.
4) The fourth section is new for HSA.  It is the device-side kernel timestamp
   section, showing a list of each kernel dispatched by the application.  For
   each kernel, the following information is shown:
   a) the name of the kernel symbol associated with the kernel dispatched (or
      "<UnknownKernelName>" if the profiler could not extract the symbol name.)
   b) the kernel pointer value of the kernel dispatched.
   c) the start timestamp of the kernel, indicating when the kernel started
      executing on the device. The timestamps represent nanoseconds.
   d) the end timestamp of the kernel, indicating when the kernel finished
      executing on the device.  The timestamps represent nanoseconds.
   e) the duration of the kernel (in nanoseconds).
   f) the agent handle of the agent the kernel was dispatched to.

Several other sprofile switches will also work with the --hsatrace switch:

  1) You can control the name and location of the .atp file using sprofile's
     --outputfile switch.
  2) You can generate a subset of the Trace Summary pages using sprofile's
     --tracesummary and --atpfile switch.  For instance:
	     ./sprofile --tracesummary --atpfile myapp.atp
	 will generate an API summary, a kernel summary and a top-ten kernel
	 dispatch list from the data contained in the myapp.atp file.
  3) The following switches should also work:
       --envvar, --envvarfile, --fullenv, --sessionname --workingdirectory,
	   --interval, --maxapicalls
  4) The following switches are either not implemented yet or are not applicable
     to the HSA Profiler:
	   --apifilterfile, --nocollapse, --ret, --sym

You can get more information on the switches mentioned above by reading the
CodeXL User Guide or by executing:

  ./sprofile --hsatrace --help

You can load an HSA .atp file into CodeXL 1.7 in order to see the application
timeline.  At this time, CodeXl 1.7 is not yet released publicly and CodeXL 1.6
does not support loading HSA .atp files.  Even though CodeXL 1.7 can load HSA .atp
files, there will be some UI issues.  It is expected that a future version of CodeXL
will fully support the HSA Profiler.

In order to profile, you will need to make sure that both the HSA runtime
libraries (libhsa-runtime64.so, libhsa-runtime-ext64.so, and libhsakmt.so)
and the HSA runtime tools library (libhsa-runtime-tools64.so) can be found
and loaded by the application you are profiling.  It is recommended that the
LD_LIBRARY_PATH environment variable contains the following two directories:
  -- The location of libhsa-runtime-tools64.so (i.e. HSA-Profiler-AMD/lib)
  -- The location of libhsa-runtime64.so (i.e. HSA-Runtime-AMD/lib)

Note: the runtime tools library (libhsa-runtime-tools64.so) is tightly
      coupled to the runtime library (libhsa-runtime64.so).  It is
      expected that if you receive an update to one of them, you will
      need to also get an update to the other.

You can profile the vector_copy sample contained in HSA-Runtime-AMD/sample
using the following steps:
  -- Build the vector_copy sample using make
  -- Verify that the sample executable runs
  -- Execute "./sprofile --hsatrace vector_copy"

