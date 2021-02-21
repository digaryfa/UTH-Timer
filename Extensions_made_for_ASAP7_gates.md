# Extensions made on TAU 2020 DCTK to support all ASU ASAP7 gates

## Introduction

In order to support all gates provided by the ASU ASAP7 PDK (INVBUF, SIMPLE, AO, OA, SEQ), we had to make several extensions on TAU 2020 contest DCTK. We made code developements and modifications for representing, storing and analyzing all kind of timing arcs of ASU gates.

## General
* Support multiple input/output gates.
* Parse and store all combinational, sequential and asynchronous timing arcs. Support for all when, timing_sense. (positive_unate, negative_unate, non_unate) and timing_type (combinational, combinational_fall, combinational_rise, rising_edge, falling_edge, preset, clear) attributes. 
* Parse and store pin-level receiver capacitances (e.g. C1,C2 capacitance on "D" pin of flip-flops).
* Parse and store ff, latch and statetable liberty groups for sequential cells.
* Extend YAML to include stages consisting of driver, receiver timing arcs with random behavior (i.e. when, timing_sense, timing_type), connected with a pi-model.

## SPICE simulation

* Extend SPICE decks to support all multiple input/output gates, all timing arcs and receiver gates without timing arc.
* Added option for SPECTRE simulator.
* Preprocessing of Xyce SPICE decks to fix "singular matrix" & "failed transient analysis" issues. Xyce preprocessing is performed to connect resistors (e.g. 100GOhm) to ground for all nodes that do not have any path to ground.
* Modified .tran command from ``'.tran <timestep> <duration> <start_time> <max_timestep>' to '.tran <timestep> <duration>' ``, which significantly speedups SPICE simulation runtime, maintainng almost the same accuracy.

Combinational and Sequential Cells:

* Tie side inputs to non controlling values for simple combinational gates that do have a "when" attribute in YAML.
* If a when attribute is specified in YAML, tie side inputs as specified by this attribute.

Only for Sequential Cells:

* flip-flop and latch outputs must be initialized before measuring delay and output slew for a specific transition. Thus, the input is set to the opposite value (e.g. to 0 if we measure 0->1 output transition) and an active clock edge is applied to initialize the output. Then, another active clock edge is applied to measure the delay and output slew.
   
* Preset and clear timing arcs are also handled after setting the correct state as mentioned above.

## CCS Delay Calculation

* Adjusted our CCS delay calculator submitted to TAU 2020 contest to handle all timing arcs. 
* Find the correct input pin, output pin, and timing arcs for driver and receiver, and calculate the corresponding driver timing arc delay and output slew.

* Account for input-only receiver cells that do not have a timing arc in YAML. Include extra interpolation techniques (e.g. interpolation for 1D receiver capacitance LUTs at the input pin level).

## ASU PDK and libs

* Generated and provided all postprocessed liberty and SPICE files for the ASU ASAP7 PDK

