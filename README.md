      ___   ____   ___  ____   _       ____  ____     ___
     /   \ |    \ /  _]|    \ | |     /    ||    \   /  _]
    |     ||  o  )  [_ |  _  || |    |  o  ||  _  | /  [_
    |  O  ||   _/    _]|  |  || |___ |     ||  |  ||    _]
    |     ||  | |   [_ |  |  ||     ||  _  ||  |  ||   [_
     \___/ |__| |_____||__|__||_____||__|__||__|__||_____|

# OpenLANE
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fefabless%2Fopenlane.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fefabless%2Fopenlane?ref=badge_shield) [![Documentation Status](https://readthedocs.org/projects/openlane/badge/?version=master)](https://openlane.readthedocs.io/en/master/?badge=master) ![CI](https://github.com/efabless/openlane/workflows/CI/badge.svg?branch=main)

This documentation is also available at ReadTheDocs [here](https://openlane.readthedocs.io/).

# Table of contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
    - [Installation Notes](#installation-notes)
- [Updating OpenLANE](#updating-openlane)
- [Setting up OpenLANE](#setting-up-openlane)
    - [Pulling the OpenLANE Docker Container](#pulling-the-openlane-docker-container)
    - [Running OpenLANE](#running-openlane)
    - [Command line arguments](#command-line-arguments)
    - [Adding a design](#adding-a-design)
- [OpenLANE Architecture](#openlane-architecture)
    - [OpenLANE Design Stages](#openlane-design-stages)
    - [OpenLANE Output](#openlane-output)
    - [Flow configuration](#flow-configuration)
- [Regression And Design Configurations Exploration](#regression-and-design-configurations-exploration)
- [Hardening Macros](#hardening-macros)
- [Chip Integration](#chip-integration)
- [Commands and Configurations](#commands-and-configurations)
- [How To Contribute](#how-to-contribute)
- [Authors](#authors)
- [Additional Material](#additional-material)
    - [Papers](#papers)
    - [Videos And Tutorials](#videos-and-tutorials)

# Overview

OpenLANE is an automated RTL to GDSII flow based on several components including OpenROAD, Yosys, Magic, Netgen, Fault, OpenPhySyn, CVC, SPEF-Extractor, CU-GR, Klayout and custom methodology scripts for design exploration and optimization. The flow performs full ASIC implementation steps from RTL all the way down to GDSII - this capability will be released in the coming weeks with completed SoC design examples that have been sent to SkyWater for fabrication.

Join the community on [slack](https://invite.skywater.tools)!

To use the latest stable release of OpenLane, please go [here](https://github.com/efabless/openlane/releases/).

# Prerequisites

 - Docker (ensure docker daemon is running) -- tested with version 19.03.12, but any recent version should suffice

# Quick Start:
You can start setting up the skywater-pdk and openlane by running:

```bash
    git clone https://github.com/efabless/openlane.git
    cd openlane/
    make openlane
    # Default PDK_ROOT is $(pwd)/pdks. If you want to install the PDK at a differnt location, uncomment the next line.
    #export PDK_ROOT=<absolute path to where skywater-pdk and open_pdks will reside>
    make pdk
    make test # This is to test that the flow and the pdk were properly installed
```

Note that `make openlane` always pulls the **latest** version of openlane: to get a specific tag, you need to invoke `IMAGE_NAME=efabless/openlane:v0.18 make openlane`, for example.

This should produce a clean run for the spm. The final layout will be generated here: `./designs/spm/runs/openlane_test/results/magic/spm.gds`.

To run the regression test, which tests the flow against all available designs under [./designs/](./designs/) vs the the benchmark results, run the following command:

```bash
    make regression_test
```

Your results will be compared with: [sky130_fd_sc_hd](https://github.com/efabless/openlane/blob/master/regression_results/benchmark_results/SW_HD.csv).

After running you'll find a directory added under [./regression_results/](./regression_results) it will contain all the reports needed for you to know whether you've been successful or not. Check [this](./regression_results/README.md#output) for more details.

**Note**: if `flow_status` is `flow_failed`, that means the design failed. Any reported statistics from any run after the failure of the design is reported as `-1` as well.

Now you can skip forward to [running openlane](#running-openlane).

## Installation Notes:

- The Makefile should do the following when you run the above command:
    - Clone Skywater-pdk and the specified STD_CELL_LIBRARY, SPECIAL_VOLTAGE_LIBRARY, and IO_LIBRARY and build it.
    - Clone open_pdks and set up the pdk for OpenLANE use.
    - Pull the OpenLANE docker container.
    - Test the whole setup with a complete run on a small design `spm`.

- the default STD_CELL_LIBRARY is sky130_fd_sc_hd. You can change that by running:
```bash
    export STD_CELL_LIBRARY=<Library name, i.e. sky130_fd_sc_ls>
```
- Other options are:
    - sky130_fd_sc_hs
    - sky130_fd_sc_ms
    - sky130_fd_sc_ls
    - sky130_fd_sc_hdll

- You can install the full pdk by running `make full-pdk` instead of `make pdk`

- You can install the pdk manually -outside of the Makefile- by following the instructions provided [here][30].

- Refer to [this][24] for more details on the structure.

- For curious users: For more details about the docker container and its process, the [following instructions][1] walk you through the process of using docker containers to build the needed tools then integrate them into OpenLANE flow. **You Don't Need To Re-Build It.**

# Updating OpenLANE

If you already have the repo locally, then no need to re-clone it. You can directly run the following:

```bash
    cd openlane/
    git checkout master
    git pull
    export PDK_ROOT=<absolute path to where skywater-pdk and open_pdks will reside>
    make openlane
    make pdk
    make test # This is to test that the flow and the pdk were properly installed
```

This should install the latest openlane docker container, and re-install the pdk for the latest used version.


# Setting up OpenLANE

## Pulling the OpenLANE Docker Container

**DISCLAIMER: This sub-section is to give you an understanding of what happens under the hood in the Makefile. You don't need to run the instructions here, if you already ran `make openlane`**

To setup openlane you can pull the docker container following these instructions:

```bash
    git clone https://github.com/efabless/openlane.git
    docker pull efabless/openlane:current
```


## Running OpenLANE

### Starting The Docker Container

You have one of two options:


The easiest way to mount the proper directories into the docker container would be to rely on the Makefile:

```bash
    make mount
```

* **Note:**
    - Default PDK_ROOT is `$(pwd)/pdks`. If you have installed the PDK at a different location, run the following before `make mount`:
        ```bash
        export PDK_ROOT=<absolute path to where skywater-pdk, open_pdks, and sky130A reside>
        ```
    - Default IMAGE_NAME is efabless/openlane:current. If you want to use a different version, run the following before `make mount`:
        ```bash
        export IMAGE_NAME=<docker image name>
        ```

The following is roughly what happens under the hood when you run `make mount` + the required exports:

```bash
    export PDK_ROOT=<absolute path to where skywater-pdk and open_pdks will reside>
    export IMAGE_NAME=<docker image name>
    docker run -it -v $(pwd):/openLANE_flow -v $PDK_ROOT:$PDK_ROOT -e PDK_ROOT=$PDK_ROOT -u $(id -u $USER):$(id -g $USER) $IMAGE_NAME
```

**Note: this will mount the openlane directory and the pdk root inside the container.**

### Running inside the Docker Container

Use the following example to check the overall setup:

```bash
./flow.tcl -design spm
```

To run OpenLANE on multiple designs at the same time, check this [section](#regression-and-design-configurations-exploration).

Having trouble running the flow? check [FAQs](https://github.com/efabless/openlane/wiki)

## Command line arguments

The following are arguments that can be passed to `flow.tcl`

<table>
    <tr>
        <th width="196">
        Argument
        </th>
        <th >
        Description
        </th>
    </tr>
    <tr>
        <td align="center">
            <code>-design &lt;folder path&gt;</code> <br> (Required)
        </td>
        <td align="justify">
            Specifies the design folder. A design folder should contain a config.tcl defining the design parameters. <br> If the folder is not found, ./designs directory is searched
        </td>
    </tr>
    <tr>
        <td align="center">
            <code>-config_file &lt;file&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Specifies the design's configuration file for running the flow. <br> For example, to run the flow using <code>/spm/config2.tcl</code> <br> Use run <code>./flow.tcl -design /spm -config_file /spm/config2.tcl</code> <br> By default <code>config.tcl</code> is used.
        </td>
    </tr>
        <tr>
        <td align="center">
            <code>-config_tag &lt;name&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Specifies the design's configuration file for running the flow. <br> For example, to run the flow using <code>designs/spm/config2.tcl</code> <br> Use run <code>./flow.tcl -design spm -config_tag config2</code> <br> By default <code>config</code> is used.
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-tag &lt;name&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Specifies a <code>name</code> for a specific run. If the tag is not specified, a timestamp is generated for identification of that run. <br> Can Specify the configuration file name in case of using <code>-init_design_config</code>
        </td>
    </tr>
        <tr>
        </tr>
        <td align="center">
            <code>-run_path &lt;path&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Specifies a <code>path</code> to save the run in. By default the run is in <code>design_path/</code>, where the design path is the one passed to <code>-design</code>
        </td>
    </tr>
        <tr>
        </tr>
        <td align="center">
            <code>-save <br> (Optional)
        </td>
        <td align="justify">
            A flag to save a runs results like .mag and .lef in the design's folder
        </td>
    </tr>
        <tr>
        </tr>
        <td align="center">
            <code>-save_path &lt;path&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Specifies a different path to save the design's result. This options is to be used with the <code>-save</code> flag
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-src &lt;verilog_source_file&gt; </code> <br> (Optional)
        </td>
        <td td align="justify">
            Sets the verilog source code file(s) in case of using `-init_design_config`. <br> The default is that the source code files are under <code>design_path/src/</code>, where the design path is the one passed to <code>-design</code>
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-init_design_config </code> <br> (Optional)
        </td>
        <td td align="justify">
            Creates a tcl configuration file for a design. <code>-tag &lt;name&gt;</code> can be added to rename the config file to <code>&lt;name&gt;.tcl</code>
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-overwrite</code> <br> (Optional)
        </td>
        <td align="justify">
            Flag to overwirte an existing run with the same tag
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-interactive</code> <br> (Optional)
        </td>
        <td align="justify">
            Flag to run openlane flow in interactive mode
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-file &lt;file_path&gt;</code> <br> (Optional)
        </td>
        <td align="justify">
            Passes a script of interactive commands in interactive mode
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-synth_explore</code> <br> (Boolean)
        </td>
        <td align="justify">
            If enabled, synthesis exploration will be run (only synthesis exploration), which will try out the available synthesis strategies against the input design. The output will be the four possible gate level netlists under &lt;run_path/results/synthesis&gt; and a summary report under reports that compares the 4 outputs.
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-lvs</code> <br> (Boolean)
        </td>
        <td align="justify">
            If enabled, only LVS will be run on the design. in which case the user must also pass: -design DESIGN_DIR -gds DESIGN_GDS -net DESIGN_NETLIST.
        </td>
    </tr>
    <tr>
        </tr>
        <td align="center">
            <code>-drc</code> <br> (Boolean)
        </td>
        <td align="justify">
            If enabled, only DRC will be run on the design. in which case the user must also pass: -design DESIGN_DIR -gds DESIGN_GDS -report OUTPUT_REPORT_PATH -magicrc MAGICRC.
        </td>
    </tr>
</table>


## Adding a design

To add a new design, follow the instructions provided [here](./designs/README.md)

This [file](./designs/README.md) also includes useful information about the design configuration files. It also includes useful utilities for exploring and updating design configurations for each (PDK,STD_CELL_LIBRARY) pair.

# OpenLANE Architecture

<table>
  <tr>
    <td  align="center"><img src="./docs/_static/openlane.flow.1.png" ></td>
  </tr>

</table>


## OpenLANE Design Stages

OpenLANE flow consists of several stages. By default all flow steps are run in sequence. Each stage may consist of multiple sub-stages. OpenLANE can also be run interactively as shown [here][25].

1. **Synthesis**
    1. `yosys` - Performs RTL synthesis
    2. `abc` - Performs technology mapping
    3. `OpenSTA` - Pefroms static timing analysis on the resulting netlist to generate timing reports
2. **Floorplan and PDN**
    1. `init_fp` - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
    2. `ioplacer` - Places the macro input and output ports
    3. `pdn` - Generates the power distribution network
    4. `tapcell` - Inserts welltap and decap cells in the floorplan
3. **Placement**
    1. `RePLace` - Performs global placement
    2. `Resizer` - Performs optional optimizations on the design
    3. `OpenPhySyn` - Performs timing optimizations on the design
    4. `OpenDP` - Perfroms detailed placement to legalize the globally placed components
4. **CTS**
    1. `TritonCTS` - Synthesizes the clock distribution network (the clock tree)
5. **Routing**
    1. `FastRoute` - Performs global routing to generate a guide file for the detailed router
    2. `CU-GR` - Another option for performing global routing.
    3. `TritonRoute` - Performs detailed routing
    4. `SPEF-Extractor` - Performs SPEF extraction
6. **GDSII Generation**
    1. `Magic` - Streams out the final GDSII layout file from the routed def
    2. `Klayout` - Streams out the final GDSII layout file from the routed def as a back-up
7. **Checks**
    1. `Magic` - Performs DRC Checks & Antenna Checks
    2. `Klayout` - Performs DRC Checks
    3. `Netgen` - Performs LVS Checks
    4. `CVC` - Performs Circuit Validity Checks

OpenLANE integrated several key open source tools over the execution stages:
- RTL Synthesis, Technology Mapping, and Formal Verification : [yosys + abc][4]
- Static Timing Analysis: [OpenSTA][8]
- Floor Planning: [init_fp][5], [ioPlacer][6], [pdn][16] and [tapcell][7]
- Placement: [RePLace][9] (Global), [Resizer][15] and [OpenPhySyn][28] (Optimizations), and [OpenDP][10] (Detailed)
- Clock Tree Synthesis: [TritonCTS][11]
- Fill Insertion: [OpenDP/filler_placement][10]
- Routing: [FastRoute][12] or [CU-GR][36] (Global) and [TritonRoute][13] (Detailed)
- SPEF Extraction: [SPEF-Extractor][27]
- GDSII Streaming out: [Magic][14] and [Klayout][35]
- DRC Checks: [Magic][14] and [Klayout][35]
- LVS check: [Netgen][22]
- Antenna Checks: [Magic][14]
- Circuit Validity Checker: [CVC][31]

## OpenLANE Output

All output run data is placed by default under ./designs/design_name/runs. Each flow cycle will output timestamp-marked foler containing the following file structure:

```
designs/<design_name>
├── config.tcl
├── runs
│   ├── <tag>
│   │   ├── config.tcl
│   │   ├── logs
│   │   │   ├── cts
│   │   │   ├── cvc
│   │   │   ├── floorplan
│   │   │   ├── klayout
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── reports
│   │   │   ├── cts
│   │   │   ├── cvc
│   │   │   ├── floorplan
│   │   │   ├── klayout
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── results
│   │   │   ├── cts
│   │   │   ├── cvc
│   │   │   ├── floorplan
│   │   │   ├── klayout
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   └── tmp
│   │       ├── cts
│   │       ├── cvc
│   │       ├── floorplan
│   │       ├── klayout
│   │       ├── magic
│   │       ├── placement
│   │       ├── routing
│   │       └── synthesis
```

To delete all generated runs under all designs:
- inside the docker container:
    ```bash
        ./clean_runs.tcl
    ```
- outside the docker container:
    ```bash
        make clean_runs
    ```

## Flow configuration

1. PDK / technology specific
2. Flow specific
3. Design specific

- A PDK should define at least one standard cell library(SCL) for the PDK. A common configuration file for all SCLs is located in:

    ```
    $PDK_ROOT/$PDK/config.tcl
    ```

    - Sometimes the PDK comes with several standard cell libraries. Each has an own configuration file that defines extra variables specific to the SCL. It may also override variables in the common PDK configuration file which is located in:

        ```
        $PDK_ROOT/$PDK/$STD_CELL_LIBRARY/config.tcl
        ```
    - More on configuring a new PDK in this [section](#setting-up-the-pdk-skywater-pdk)

- Flow specific variables are related to the flow and are initialized with default values in:

    ```
    ./configuration/
    ```

- Finally, each design should have it's own configuration file with some required variables which are available in this [list][17]. A design configuration file may override any of the variables defined in PDK or flow configuration files. This is the global configurations for the design:

    ```
    ./designs/<design>/config.tcl
    ```
    - More on design configurations in [here](./designs/README.md)

A list of all available variables can be found [here][17].



# Regression And Design Configurations Exploration

As mentioned earlier, everytime a new design or a new (PDK,STD_CELL_LIBRARY) pair is added, or any update happens in the flow tools, a re-configuration for the designs is needed. The reconfiguration is methodical and so an exploration script was developed to aid the designer in reconfiguring his designs if needed.
As explained [here](#adding-a-design) that each design has multiple configuration files for each (PDK,STD_CELL_LIBRARY) pair.

OpenLANE provides `run_designs.py`, a script that can do multiple runs in a parallel using different configurations. A run consists of a set of designs and a configuration file that contains the configuration values. It is useful to explore the design implementation using different configurations to figure out the best one(s).

Also, it can be used for testing the flow by running the flow against several designs using their best configurations. For example the following run: spm using its default configuration files `config.tcl.` :
```
python3 run_designs.py --designs spm xtea md5 aes256 --tag test --threads 3
```

For more information on how to run this script, refer to this [file][21]

For more information on design configurations, how to update them, and the need for an exploration for each design, refer to this [file](./designs/README.md)

# Hardening Macros

This is discussed in more detail [here][29].

# Chip Integration

The first step of chip integration is hardening the macros. To learn more about this check this [file][29].

Using openlane, you can produce a GDSII from a chip RTL. This is done by applying a certain methodology that we follow using our custom scripts and the integrated tools.

To learn more about Chip Integration. Check this [file][26]

# Commands and Configurations

To get a full list of the openlane commands, first introduce yourself to the interactive mode of openlane [here][25]. Then check the full documentation of the OpenLANE commands [here][34].

The full documentation of OpenLANE run configurations could be found [here][2].
# How To Contribute

We discuss the details of how to contribute to OpenLANE in [this documentation][32].

# Authors

To check the original author list of OpenLANE, check [this][33].

# Additional Material

## Papers

- Ahmed Ghazy and Mohamed Shalan, "OpenLane: The Open-Source Digital ASIC Implementation Flow", Article No.21, Workshop on Open-Source EDA Technology (WOSET), 2020. [Paper](https://github.com/woset-workshop/woset-workshop.github.io/blob/master/PDFs/2020/a21.pdf)
- M. Shalan and T. Edwards, "Building OpenLANE: A 130nm OpenROAD-based Tapeout- Proven Flow : Invited Paper," 2020 IEEE/ACM International Conference On Computer Aided Design (ICCAD), San Diego, CA, USA, 2020, pp. 1-6.
- R. Timothy Edwards, M. Shalan and M. Kassem, "Real Silicon using Open Source EDA," in IEEE Design & Test, doi: 10.1109/MDAT.2021.3050000.

## Videos and Tutorials

### OpenLane Specific

- [FOSSi Dial-Up - OpenLane, A Digital ASIC Flow for SkyWater 130nm Open PDK, Mohamed Shalan](https://www.youtube.com/watch?v=Vhyv0eq_mLU)
- [Openlane Overview, Ahmed Ghazy](https://www.youtube.com/watch?v=d0hPdkYg5QI)
- [Free VLSI Tutorial - VSD - A complete guide to install Openlane and Sky130nm PDK](https://www.udemy.com/course/vsd-a-complete-guide-to-install-openlane-and-sky130nm-pdk)
- [Sky130 - Exploring OpenLANE and OpenDB to create a register file , Sylvain Munaut](https://www.youtube.com/watch?v=AT_LcmaCZmw)
- [VLSI SoC EDA openLANE with Skywater 130 PDK, Gary Huang](https://www.youtube.com/watch?v=QnJzoJjC7RQ)

### Caravel & SkyWater PDK
- [Aboard Caravel, Ahmed Ghazy](https://www.youtube.com/watch?v=9QV8SDelURk)
- [FOSSi Dial-Up - Skywater PDK: Fully open source manufacturable PDK for a 130nm process, Tim Ansell](https://www.youtube.com/watch?v=EczW2IWdnOM&)
- [Skywater 130nm PDK - Initial Discovery, Sylvain Munaut](https://www.youtube.com/watch?v=gRYBdTXbxiU)

[1]: ./docker_build/README.md
[2]: ./configuration/README.md
[4]: https://github.com/YosysHQ/yosys
[5]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/init_fp
[6]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/openroad/src/ioPlacer
[7]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/openroad/src/tapcell
[8]: https://github.com/The-OpenROAD-Project/OpenSTA
[9]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/openroad/src/replace
[10]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/openroad/src/opendp
[11]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/TritonCTS
[12]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/openroad/src/FastRoute
[13]: https://github.com/The-OpenROAD-Project/TritonRoute
[14]: https://github.com/RTimothyEdwards/magic
[15]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/openroad/src/resizer
[16]: https://github.com/The-OpenROAD-Project/OpenROAD/tree/openroad/src/pdngen
[17]: ./configuration/README.md
[18]: https://github.com/RTimothyEdwards/qflow/blob/master/src/addspacers.c
[19]: https://github.com/The-OpenROAD-Project/
[20]: https://github.com/git-lfs/git-lfs/wiki/Installation
[21]: ./regression_results/README.md
[22]: https://github.com/RTimothyEdwards/netgen
[24]: ./docs/source/PDK_STRUCTURE.md
[25]: ./docs/source/advanced_readme.md
[26]: ./docs/source/chip_integration.md
[27]: https://github.com/HanyMoussa/SPEF_EXTRACTOR
[28]: https://github.com/scale-lab/OpenPhySyn
[29]: ./docs/source/hardening_macros.md
[30]: ./docs/source/Manual_PDK_installation.md
[31]: https://github.com/d-m-bailey/cvc
[32]: ./CONTRIBUTING.md
[33]: ./AUTHORS.md
[34]: ./docs/source/OpenLANE_commands.md
[35]: https://github.com/KLayout/klayout
[36]: https://github.com/cuhk-eda/cu-gr 

Run 'picorv32a' design synthesis using OpenLANE flow and generate necessary outputs.
Commands to invoke the OpenLANE flow and perform synthesis

```
# Change directory to openlane flow directory
cd Desktop/work/tools/openlane_working_dir/openlane

# alias docker='docker run -it -v $(pwd):/openLANE_flow -v $PDK_ROOT:$PDK_ROOT -e PDK_ROOT=$PDK_ROOT -u $(id -u $USER):$(id -g $USER) efabless/openlane:v0.21'
# Since we have aliased the long command to 'docker' we can invoke the OpenLANE flow docker sub-system by just running this command
docker
# Now that we have entered the OpenLANE flow contained docker sub-system we can invoke the OpenLANE flow in the Interactive mode using the following command
./flow.tcl -interactive

# Now that OpenLANE flow is open we have to input the required packages for proper functionality of the OpenLANE flow
package require openlane 0.9

# Now the OpenLANE flow is ready to run any design and initially we have to prep the design creating some necessary files and directories for running a specific design which in our case is 'picorv32a'
prep -design picorv32a

# Now that the design is prepped and ready, we can run synthesis using following command
run_synthesis

# Exit from OpenLANE flow
exit

# Exit from OpenLANE flow docker sub-system
exit

```
Calculate the die area in microns from the values in floorplan def.

Die width in unit distance = 660685 − 0 = 660685

Load generated floorplan def in magic tool and explore the floorplan.
Commands to load floorplan def in magic in another terminal
Die width in unit distance = 660685 − 0 = 660685

Die height in unit distance = 671405 − = 671405

Distance in microns = Value in Unit Distance / 1000

Die width in microns = 660685 / 1000 = 660.685 Microns 

Die height in microns = 671405 / 1000 = 671.405 Microns 

Area of die in microns = 660.685 * 671.405 = 443587.212425 Square Microns 

Load generated floorplan def in magic tool and explore the floorplan.

Commands to load floorplan def in magic in another terminal

```
# Change directory to path containing generated floorplan def
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/17-03_12-06/results/floorplan/

# Command to load the floorplan def in magic tool
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &

```
Screenshots of floorplan def in magic

<img width="1364" height="674" alt="dcap cells" src="https://github.com/user-attachments/assets/e0adf6c5-d5db-4418-9e04-b9ac584566c6" />


Equidistant placement of ports


Decap Cells and Tap Cells


Design library cell using Magic Layout and ngspice characterization

```
clone inverter standard cell design from github  
https://github.com/nickson-jose/vsdstdcelldesign 
```

Load the custom inverter layout in magic and analyse 
Spice extraction of inverter in magic 
Editing the spice model file for analysis through simulation 
Post-layout ngspice simulations
Resolve drc errors by simulation and tech file analysis of skywater pdk 


```
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Clone the repository with custom inverter design
git clone https://github.com/nickson-jose/vsdstdcelldesign

# Change into repository directory
cd vsdstdcelldesign

# Copy magic tech file to the repo directory for easy access
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .

# Check contents whether everything is present
ls

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &

```

<img width="864" height="558" alt="extract parasitic capacitance" src="https://github.com/user-attachments/assets/86039ca0-c5f4-41cf-80bf-7b9f270cd348" />

<img width="1366" height="672" alt="floor_plan_run_command1" src="https://github.com/user-attachments/assets/4b7abdb0-5023-4f7f-ad3c-ba7d4580351d" />

<img width="1366" height="678" alt="floorplan_run1" src="https://github.com/user-attachments/assets/521e7782-35ed-45a3-9fff-1b9d14faf3b3" />

<img width="1361" height="668" alt="IO in equidistant" src="https://github.com/user-attachments/assets/0d80e931-01f8-41a6-ae7b-1f931525d103" /> 

<img width="1355" height="604" alt="labdrc1" src="https://github.com/user-attachments/assets/30d6b302-d188-4764-8fb7-04f71b274a3d" /> 

<img width="1366" height="764" alt="lef file1" src="https://github.com/user-attachments/assets/12fdf256-fc95-49ae-a028-dfd46978f571" /> 

<img width="1366" height="768" alt="lef12" src="https://github.com/user-attachments/assets/6e8a849b-072e-4918-b66b-bf03f28675e6" />

**<img width="1296" height="696" alt="locali" src="https://github.com/user-attachments/assets/4026e76a-a313-4ba6-a45d-7628d35c5193" />

<img width="1366" height="711" alt="magic T" src="https://github.com/user-attachments/assets/318b1ce8-ebc1-4fb1-b98a-68ebf7bfd9b6" />

<img width="1175" height="670" alt="run_placement1" src="https://github.com/user-attachments/assets/07247cbd-6d98-4592-9dcf-278e6bf55b68" />

<img width="1175" height="670" alt="run_placement2" src="https://github.com/user-attachments/assets/0fa10763-2cb6-4fc3-b5f6-5a71d3dfe377" />




**









Screenshot of commands run

<img width="1363" height="767" alt="1" src="https://github.com/user-attachments/assets/b343abdd-67a4-494f-9815-e35d83abd7cd" />

<img width="1366" height="709" alt="2" src="https://github.com/user-attachments/assets/dd0d9c03-d89f-4383-b30c-2de7c8d9a514" />

<img width="1366" height="709" alt="3" src="https://github.com/user-attachments/assets/a4c0d3d3-83d6-404e-89e4-def3b60f9839" />

<img width="1366" height="774" alt="4" src="https://github.com/user-attachments/assets/b3fbf6bd-98d2-469e-adcf-5762cfcee3a8" />

<img width="1366" height="711" alt="43" src="https://github.com/user-attachments/assets/97bb21c0-ae24-4432-9d42-b7a2d6769d8e" />






<img width="1366" height="716" alt="42" src="https://github.com/user-attachments/assets/f791a340-705d-4895-b239-c69dcd2d42bd" />


----


<img width="1365" height="768" alt="box2" src="https://github.com/user-attachments/assets/df321f0e-5df5-47b7-8499-54cc6454eec9" />

<img width="1366" height="768" alt="drc interactive vias style fast slow" src="https://github.com/user-attachments/assets/b174e9ee-ec24-4a05-8352-9737371d897c" />

<img width="1335" height="725" alt="drc shrink" src="https://github.com/user-attachments/assets/694a677f-c11e-4ebb-815d-bf71f89d0806" />

<img width="1362" height="748" alt="drc1" src="https://github.com/user-attachments/assets/f77420e9-79ac-40bb-bf78-c533ee7fd5bd" />

<img width="1355" height="596" alt="drc2" src="https://github.com/user-attachments/assets/7ea2d713-7ae7-4cf9-ae34-af299fc7d9c3" />

<img width="1366" height="745" alt="drc3" src="https://github.com/user-attachments/assets/c1080ad7-b7d1-4843-b0f2-ce428e3df373" />

<img width="1364" height="757" alt="drc5" src="https://github.com/user-attachments/assets/8b06a20d-1428-4a18-b7c7-0a49ccd803b6" />

<img width="1335" height="568" alt="config tcl1" src="https://github.com/user-attachments/assets/12ddca0f-ead4-43f9-a077-a4d6e120b4f5" />

<img width="1333" height="572" alt="config tcl2" src="https://github.com/user-attachments/assets/2e81821a-1bd8-49cd-9843-6ac6e141540b" />

<img width="1341" height="569" alt="cp lef to picorv32a" src="https://github.com/user-attachments/assets/8aaa1cfb-49b9-4d6d-85d8-728290ec279c" />







<img width="1365" height="680" alt="magic tech lef read" src="https://github.com/user-attachments/assets/fbdddd02-bc33-490d-9ea0-b8eff601b0ec" />

<img width="1360" height="641" alt="magic2" src="https://github.com/user-attachments/assets/b8e7b9d5-cccc-4e6e-a0d5-6caec2fc1caa" />

<img width="1364" height="732" alt="magic3" src="https://github.com/user-attachments/assets/6a7f3417-91e4-4019-9b8b-ad07052aa65f" />

<img width="1366" height="714" alt="magic4" src="https://github.com/user-attachments/assets/e0f1b682-5b5c-49ec-afe9-f46c988a238a" />

<img width="1366" height="711" alt="merge 1" src="https://github.com/user-attachments/assets/1a205bfc-f845-479e-ab08-ffd8709b7176" /> 

<img width="1295" height="544" alt="nagic1" src="https://github.com/user-attachments/assets/876e0b05-48b8-449d-b0a6-fc5b70be80e1" />

<img width="1366" height="757" alt="ng1" src="https://github.com/user-attachments/assets/0ca4e7cc-b1bc-43f2-bc49-9152b8f3e3be" />

<img width="1366" height="735" alt="ngplot1" src="https://github.com/user-attachments/assets/cdf427f7-7e3a-4570-a366-2b0d446d7e8d" />

<img width="1366" height="707" alt="ngplot2" src="https://github.com/user-attachments/assets/f24a650b-2eef-45ff-818a-83659a4572cd" />

<img width="1366" height="733" alt="ngspicegraphplot1" src="https://github.com/user-attachments/assets/cfd89b5d-a378-41fa-9d2b-b6be8fa4d63a" />

<img width="1366" height="768" alt="nwell2" src="https://github.com/user-attachments/assets/0b8c2de4-b79e-4064-850a-02405e91b04e" />

<img width="1366" height="768" alt="plot12" src="https://github.com/user-attachments/assets/5dd5a14f-cc70-4ad0-b1ff-768992f99218" />

<img width="1091" height="603" alt="portname 1" src="https://github.com/user-attachments/assets/dc118755-a7fd-4db4-bd80-5bf01ec157ac" />

<img width="1339" height="533" alt="power planning" src="https://github.com/user-attachments/assets/4b1b36a2-ec1e-416d-ad8c-c7ff5a6cc4b5" />

<img width="1366" height="750" alt="skytech130A" src="https://github.com/user-attachments/assets/e741f393-dd28-4a5d-baa5-f0d1d65a83ea" />

<img width="1350" height="602" alt="snap int box commands" src="https://github.com/user-attachments/assets/35df2f3a-7498-4299-a48d-7c72e775a3a5" />

<img width="1366" height="665" alt="tap cell prevents latchup in cmos dev they connect nwell to vdd and substrate to gnd to prevent latch up" src="https://github.com/user-attachments/assets/c503ffee-6f79-4a71-9076-ad6c07c77624" />

<img width="1366" height="718" alt="test1" src="https://github.com/user-attachments/assets/89fe56ed-6d02-43aa-ac5f-ddd2cf7f396f" />

<img width="1364" height="661" alt="tkcon what 1" src="https://github.com/user-attachments/assets/7f39e6ff-9bb2-4c87-b2a3-5f2b462afd77" />

<img width="1350" height="604" alt="tracks info" src="https://github.com/user-attachments/assets/6d838a3b-0137-4223-92ea-6fdb168b897e" />

<img width="1356" height="677" alt="tracks info1" src="https://github.com/user-attachments/assets/970bb947-5904-4ccb-8778-5d75ca2c634a" />

<img width="1366" height="767" alt="cif see VIA2" src="https://github.com/user-attachments/assets/2c694a53-0be5-40ba-ab6e-bc9982cb1d51" />

<img width="1366" height="697" alt="vsd2" src="https://github.com/user-attachments/assets/4ca2b7a0-b63d-423a-a023-3e5590e35b2a" />

<img width="1366" height="711" alt="vsd3" src="https://github.com/user-attachments/assets/0b5e5d1a-06d7-4756-9910-94489c48db05" />

<img width="1306" height="605" alt="vsd11" src="https://github.com/user-attachments/assets/e16c38f6-03d4-46f8-be9b-8b1db9a452e0" />

<img width="1366" height="730" alt="vsdstdcelldesign sky130_inv" src="https://github.com/user-attachments/assets/77f4a38b-954e-4b6f-bc18-f8d658610f7f" />

<img width="1362" height="711" alt="vsdstdcelldesign_inverter_spice" src="https://github.com/user-attachments/assets/46a957d0-df59-4b22-9ec0-237bf669a539" />


<img width="1366" height="709" alt="chip area for module1" src="https://github.com/user-attachments/assets/3922c2e4-031c-4c8e-9104-9e8aa3481392" />

<img width="1332" height="675" alt="error_run_synth1" src="https://github.com/user-attachments/assets/5b835676-e476-4796-b7a5-be66cb066df0" />

<img width="1366" height="727" alt="init_floorplan" src="https://github.com/user-attachments/assets/0d34aead-4502-4496-b12f-ff687df2f176" />


<img width="1366" height="711" alt="metal1_rail_alignment" src="https://github.com/user-attachments/assets/82612ef8-27de-451e-9e64-27bfbf8dddaa" />

<img width="1366" height="735" alt="metal1_rail_alignment1" src="https://github.com/user-attachments/assets/26f416d6-056e-49a9-9e96-a07933c34fd6" />

<img width="1366" height="718" alt="sky130_vsdinv1" src="https://github.com/user-attachments/assets/164a70a6-e43b-40d6-9e77-e26fcae1cc07" />

<img width="1366" height="715" alt="sky139_vsdinv" src="https://github.com/user-attachments/assets/c5f714b2-5383-4e8e-8bfc-950f412d26cc" />

<img width="1359" height="680" alt="spice1_graph_plot1" src="https://github.com/user-attachments/assets/18ae5cd5-4131-480b-99a7-39f1bb6653df" />

<img width="1366" height="709" alt="tap_decap_or" src="https://github.com/user-attachments/assets/be73eda8-0403-4247-a695-87de09436ebc" />

<img width="1361" height="714" alt="test1" src="https://github.com/user-attachments/assets/5cd93bb5-17bc-4c06-a282-73d2c097043d" />

<img width="1366" height="737" alt="test2" src="https://github.com/user-attachments/assets/b5bd948e-be67-4134-bf7d-4e2eee348916" />

<img width="1175" height="738" alt="write_lef" src="https://github.com/user-attachments/assets/4d939dd0-d6ec-4d22-85a9-528b7178892c" />

<img width="1178" height="701" alt="write_lef1" src="https://github.com/user-attachments/assets/7ce82aa2-d4c8-4c4c-b90a-1d9a0a08879f" />

<img width="1366" height="709" alt="tap_decap_or" src="https://github.com/user-attachments/assets/4a213bfa-3a21-4260-a43c-8dd36699917d" />

<img width="1366" height="727" alt="init_floorplan" src="https://github.com/user-attachments/assets/42d7fa98-743a-4132-a810-d020e15c2531" />

<img width="1363" height="729" alt="pre_sta1" src="https://github.com/user-attachments/assets/759df6e6-be96-4038-bea0-d91231e19c17" />

<img width="1366" height="729" alt="pre_sta2" src="https://github.com/user-attachments/assets/418eddfb-16ae-4120-a79a-8d7bfc69bca1" />

<img width="1366" height="735" alt="pre_sta3" src="https://github.com/user-attachments/assets/6d7ee6e1-0daa-4237-845d-c599dd77a4e6" />

<img width="1361" height="722" alt="pre_sta4" src="https://github.com/user-attachments/assets/870700e6-d262-4be6-9362-b1d8582bdc5f" />

<img width="1362" height="717" alt="run_cts" src="https://github.com/user-attachments/assets/4e8c1ce1-e049-49d5-88a6-52a083becdf4" />

<img width="1365" height="765" alt="run_cts_full1" src="https://github.com/user-attachments/assets/8d5dc7d1-4a0b-4063-bb93-63f1918d1ae9" />

<img width="1365" height="768" alt="run_cts_successfull1" src="https://github.com/user-attachments/assets/516b8ee3-6285-4cfc-b359-8d4d413672c7" />

<img width="1366" height="746" alt="run_cts1" src="https://github.com/user-attachments/assets/9b38b58a-17b9-4609-94ff-3f151a248ae8" />

<img width="1364" height="768" alt="run_cts2" src="https://github.com/user-attachments/assets/518bbc1c-c003-419f-bed0-ff690b20ee19" />

<img width="1366" height="768" alt="run_cts3" src="https://github.com/user-attachments/assets/0c2dbe4a-fb42-4334-8fbe-ec0b3124ea48" />

<img width="1366" height="767" alt="run_cts11" src="https://github.com/user-attachments/assets/331a9f7c-59ba-4826-9efa-752b7d919e1f" />










<img width="1366" height="738" alt="picorv32_placement" src="https://github.com/user-attachments/assets/a83db13b-ef7a-425f-b556-192dd5af05cc" />

<img width="1361" height="719" alt="pdn_gen" src="https://github.com/user-attachments/assets/50dd1d00-80dd-46de-8201-561e2e80ee05" />

<img width="1359" height="757" alt="pdn1" src="https://github.com/user-attachments/assets/21552530-c9a4-4a75-bc7e-6b38cfd3dafc" />




















































