# STA_benchmark

Here lists the detailed instructions for running this benchmark suite.  
There are 27 benchmarks in total. Each benchmark will be tested with pre-route STA, post-route STA. Results are dumped as json files.

## Benchmarks

- Combinational

| c1355 | c17 | c1908 | c2670 | c3540 | c3\_slack | c432 | c499 | c5315 | c6288 | c7552 | c880 |
|-------|-----|-------|-------|-------|-----------|------|------|-------|-------|-------|------|

- Sequential

| s1196 | s1494 | s27 | s344 | s349 | s386 | s400 | s510 | s526 | ac97\_ctrl | aes\_core | des\_perf | vga\_lcd |
|-------|-------|-----|------|------|------|------|------|------|------------|-----------|-----------|----------|

- Small testcases from OpenTimer/OpenSTA

| simple | example |
|--------|---------|

## File Structures
```
common_benchmark
├── c1355
│    └── *.v
│    └── *.db
│    └── *.lib
│    └── *.sdc
│    └── *.spef
│    └── *_preroute_ost.tcl
│    └── *_postroute_ost.tcl
│    └── ...
├── c17
│    └── ...
├── c1908
│    └── ...
.
sta_output
├── c17_preroute_ost.out
├── c17_postroute_ost.out
├── ...
.
toJson
├── s526_preroute_ost_read_report.json
├── s526_postroute_ost_read_report.json
.
.
.
├── testScripts
│    └── ...
├── CommandTranslator.py
├── read_report.py
├── toJson_report_checks.py
```

Each of the benchmark contains
| File                  | Description                                                   |
|-----------------------|---------------------------------------------------------------|
| \*.v                  | verilog netlist                                               |
| \*.db                 | converted timing library for PrimeTime                        |
| \*.lib                | Liberty timing library                                        |
| \*.sdc                | timing constraints                                            |
| \*.spef               | parasitics, for post-route STA                                |
| \*\_preroute\_ost.tcl | OpenSTA script for pre-route STA                              |
| \*\_postroute\_ost.tcl| OpenSTA script for post-route STA                             |
| ost\_preroute.out     | OpenSTA pre-route output of the data points in timing report  |
| ost\_postroute.out    | OpenSTA post-route output of the data points in timing report |
| ost\_preroute.json    | OpenSTA pre-route output of the data points in json format    |
| ost\_postroute.json   | OpenSTA post-route output of the data points in json format   |

## JSON Output Format

Refer to `output_format.pptx` for the output json format.

Since the .out files contain full version of timing report, we need extra steps to fetch the target data from the timing reports. 

First, run the read_report.py to extract all needed data from the output directory and add command lines to clarify the definition of each line. 

Second, run toJson_report_checks.py to translate the output data to Json format.

## Dumping SDF
The command for dumping sdf in OpenSTA is simply `write_sdf <output filename>`.  
To generate the SDF for each benchmark, remove all `get_property` and `report_checks` commands in \<benchmark\>/\*ost.tcl and add `write_sdf <output filename>` in the end.  
For example, this is the script for dumping pre-route SDF for the `c17` benchmark:
```tcl
read_liberty -min c17_Early.lib
read_liberty -max c17_Late.lib
read_verilog c17.v
link_design "c17"
read_sdc c17.sdc
write_sdf c17_preroute.sdf
quit
```
and this is for post-route SDF:
```tcl
read_liberty -min c17_Early.lib
read_liberty -max c17_Late.lib
read_verilog c17.v
link_design "c17"
read_spef c17.spef
read_sdc c17.sdc
write_sdf c17_postroute.sdf
quit
```
