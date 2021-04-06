## NEED FOR DRC
The purpose of doing DRC - Design Rule Checks, is to make sure that all the polygons and layers from the layout database follow through the manufacturing process guidelines. This follow through is ensured by running the layout against a DRC rule deck provided by the fabrication team in order for the design to meet functionality, manufacturability and avoid chip failure. If the results thus obtained fall within permissible margin set up by fabs, the design is said to be Clean, else it’s said to be ‘Dirty’ (needs to be fixed and made clean). 

![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/DRC%20Flow.jpg)

Fig. DRC Flow

### Input:
1. Rule File (SVRF / TVF)
2. Layout Database (GDSII, OASIS, Binary, ASCII)
    Binary & ASCII are properietary to Metor Graphics Corporation and only work in flat Calibre nmDRC.

### Output:
1. Run Transcript
2. DRC Results Database
3. DRC Summary Report (optional)

DRC Results Database can be ASCII (default), GDSII, OASIS, or Binary. Generally should output an ASCII DRC results database. 
When you use Calibre nmDRC-H for mask layer generation, you should output a GDSII or OASIS DRC results database, you should also specify DRC Maximum Results ALL
in this case. You should follow these guidelines because Calibre nmDRC-H requires extra internal overhead to generate mask layer results.

### Command Line Invocations:

Flat run (Calibre nmDRC)
> calibre -drc rule_file

Hierarchical run (Calibre nmDRC-H)
> calibre -drc -hier rule_file

Multithreaded hierarchical runs
> calibre -drc -hier -turbo -hyper rule_file
> calibre -drc -hier -turbo -remote host1,host2,host3 -hyper rule_file

## CALIBRE STEPS FOR DRC
Setup the server and path to Calibre. 

Click the Settings button (Spanner icon) next to Calibre Toolbar (DRC/LVS/PEX/RVE/Settings):

For the host enter: **192.168.6.50** 

For calibre give the entire path to the current tool: **$CALIBRE_HOME/bin/calibre**

In L-Edit, click Run Calibre DRC from the Calibre toolbar which will launch the GUI.

If the Load Runset File dialog box appears, you can Cancel it.

In the GUI, click the Rules button in the left Panel and enter the following info:

DRC Rules File: Browse to the Calibre rule file eg. $XFAB_CALIBRE_RUNSET/xt018_drc_runset

You can click Load to test any problem with the rule file.

Check Selection Recipe: Ignore

DRC Run Directory: Ideally **$PROJDIR/DRC**

Click Inputs and setup the inputs but the defaults should be Ok eg. GDSII/Export from Layout Viewer

Click Outputs and the defaults should be fine.

If none of the left panel buttons are red, click Run DRC

If the run was successful, save the GUI settings to a file by clicking File->Save Runset in the DRC folder created before which can be loaded the next time.

## DRC Concepts
DRC Concepts can be divided into:
1. *Layers*
   - Layer Types: can be Original/Drawn, Derived polygon, Derived Edge, Derived Error Layers. All the derived layers are outputted to the DRC Results Database. Derived Error Layers are the only true 'Error Layers'.
   EXAMPLES:
    ```
    ORIGINAL LAYER:
    LAYER M1 2                          // simple layer      
    LAYER MET 3 4 5                     // layer set

    DERIVED POLYGON LAYER:
    gate = poly and oxide

    DERIVED EDGE LAYER:
    long_metal_edge = length metal > 5

    DERIVED ERROR LAYER:
    x = INTERNAL m1 < 0.08

    ```
   - Layer Definitions: 
    Here we assign names to derived layers, these can be used in another rule file operation by referencing its name.
    ```
    p_diff = diffusion AND p_dope           // p+ diffusion
    n_tap = n_diff NOT OUTSIDE n_well       // n-tap areas
    LAYER diff 24                           // DIFFUSION layer with GDS layer number 24

    ```
2. *Layer Operations*
   Two types - Layer Constructors & Layer Selectors
   ```
    layer1 AND layer2                       // **constructs** /creates new polygons from the polygon data on both input layers.
    layer1 COINCIDENT EDGE layer2           // **selects** edges or edge segments from the first input layer that are coincident with edges from the second input layer.
    ```
3. *Rule Check Statements*
   They specify layer operations within the rule file that instantiate the resulting derived layers into the DRC (or ERC) results database.
   Syntax:
   
   rule_check_name {
        layer_definition | layer operation
        .
        .
        .
   }
    
    RuleCheck Comments:
    3 ways-     //      OR      @       OR      /* ... */
    The last way of commenting is for multi-line comments.

4. *Dimensional Check Operations*
   The dimensional check operations generate derived error layers, derived edge layers, or derived polygon layers by measuring the separation of edges on one- or two-input layers.
    
    ![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/dimensional_check_operations.png)

You can limit the number of DRC results written to the DRC results database for any given DRC rule check by using the DRC Maximum Results specification statement, or by using the MAXIMUM RESULTS parameter to the DRC Check Map specification statement. By default, this maximum result limit is 1000 per DRC rule check.

## SIMPLE RULE FILE
   ```
    //--------------------------------
    //OPTIONAL HEADER INFORMATION
    //--------------------------------
    // REQUIRED DRC SPECIFICATION STATEMENTS
    LAYOUT SYSTEM GDSII
    LAYOUT PATH “./mydesign.gds”
    LAYOUT PRIMARY top_cell
    DRC RESULTS DATABASE “../drc_results”
    
    // OPTIONAL INCLUDED RULE FILES
    INCLUDE “/home/.../.../golden_rule_file"
    
    // ONE OR MORE DRAWN LAYER DEFINITIONS
    LAYER diff 24 // DIFFUSION
    LAYER poly 5 // POLY
    LAYER metal2 9 // METAL2
    LAYER via 12 // VIA
    
    // ONE OR MORE DERIVED LAYER DEFINITIONS
    gate = poly AND diff // GATE
    sd = diff NOT gate // SOURCE-DRAIN
    
    // ONE OR MORE DRC RULECHECKS
    min_gate_length {
    @ Gate length along POLY must be >= 3 microns.
    x = INSIDE EDGE poly diff
    INTERNAL x < 3
    }
    
    ```






