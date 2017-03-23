AriesNCL v1.0
=============

**Aries** **N**etwork Performance **C**ounters Monitoring **L**ibrary

AriesNCL is a library to monitor and record network router tile performance counters on the Aries router of Cray’s Cascade/XC30 platform.

### Build

```
make libariescounters.a
```

Make sure module papi is loaded before compiling. You may also need to unload
the darshan module.

### C API

Call the function below to initialize PAPI and set up the counters. It expects
a file called 'counters.txt' in the same directory as the executable with a
newline-delimited list of counter names to record:
```
void InitAriesCounters(char *progname, int my_rank, int reporting_rank_mod,
int* AC_event_set, char*** AC_events, long long** AC_values, int* AC_event_count)
```

Start recording counters:
```
void StartRecordAriesCounters(int my_rank, int reporting_rank_mod, int*
AC_event_set, char*** AC_events, long long** AC_values, int* AC_event_count)
```

Stop recording counters. Stores counters in memory until FinalizeAriesCounters called:
```
void EndRecordAriesCounters(int my_rank, int
reporting_rank_mod, double run_time, int* AC_event_set, char*** AC_events,
long long** AC_values, int* AC_event_count)
```

Start/stop recording counters for fine-grain profiling. Does not print timestamps,
and does not store counters in memory. Instead, returns the counters to the user,
who can then choose to save them or discard them depending on if they are for an
"interesting" period.
```
void StartRecordQuietAriesCounters(int my_rank, int reporting_rank_mod, int* AC_event_set,
char*** AC_events, long long** AC_values, int* AC_event_count)
```
```
struct timestep_counters *EndRecordQuietAriesCounters(int my_rank, int reporting_rank_mod,
double run_time, int* AC_event_set, char*** AC_events, long long** AC_values, int* AC_event_count)
```

Either discard or save the counters returned by above (Return... and Put..., respectively):
```
void ReturnAriesCounters(int my_rank, int reporting_rank_mod, struct timestep_counters *counters)
```
```
void PutAriesCounters(int my_rank, int reporting_rank_mod, struct timestep_counters *counters)
```

Writes out all the counters to YAML files, cleans up memory, stops PAPI:
```
void FinalizeAriesCounters(MPI_Comm *mod16_comm, int my_rank, int reporting_rank_mod,
int* AC_event_set, char*** AC_events, long long** AC_values, int* AC_event_count)
```

### Test

The main test is test.c, which creates the executable mpitest. It is an MPI program.
To setup the variables required by the function calls look at test.c or the
following:

	int AC_event_set;
	char** AC_events;
	long long * AC_values;
	int AC_event_count;
	int numtasks;
	MPI_Comm_size(MPI_COMM_WORLD, &numtasks);
	MPI_Group mod16_group, group_world;
	MPI_Comm mod16_comm;
	int members[(numtasks-1)/16 + 1];
	int rank;
	for (rank=0; rank<((numtasks-1)/16 + 1); rank++)
	{
	members[rank] = rank*16;
	}
	MPI_Comm_group(MPI_COMM_WORLD, &group_world);
	MPI_Group_incl(group_world, ((numtasks-1)/16 + 1), members, &mod16_group);
	MPI_Comm_create(MPI_COMM_WORLD, mod16_group, &mod16_comm);

Since every node will record the same counter information, we only record it on
one rank per node (we could have up to 3 redudant records on Edison but we
cannot easily figure out how many nodes we own on a router). The
reporting_rank_mod is the number of ranks per node.

These must be run on nodes with papi support (i.e. Cray's NPU component). All of
Edison's compute nodes have this.

Running the test will output YAML files mpitest.nettiles.1.bin and
mpitest.proctiles.1.bin, containing the counter data for NIC tiles and processor tiles,
respectively.

Other tests demonstrating repeated calls to collect counters are in test2.c and test3.c.
The first one demonstrates how to call the "Quiet" functions and save or discard counters.
The second demonstrates repeated calls to the normal functions (which save counters by default).

### Release

Copyright (c) 2014, Lawrence Livermore National Security, LLC.
Produced at the Lawrence Livermore National Laboratory.

Written by:
```
    Dylan Wang <dylan.wang@gmail.com>
    Staci Smith <smiths949@email.arizona.edu>
    Abhinav Bhatele <bhatele@llnl.gov>
```

LLNL-CODE-678960. All rights reserved.

This file is part of AriesNCL. For details, see:
https://github.com/LLNL/ariesncl.
Please also read the LICENSE file for our notice and the LGPL.
