## DRC
### Introduction
Design Rule Check (DRC) is a part of EDA tools whose purpose is to make sure that all the polygons and layers from the layout database follow through the manufacturing process guidelines recommended by foundry for reliable and fault free production of the chips. This follow through is ensured by running the layout against a DRC rule deck in order for the design to meet functionality, manufacturability and avoid chip failure. If the results thus obtained fall within permissible margin set up by fabs, the design is said to be Clean, else it’s said to be ‘Dirty’ (needs to be fixed and made clean).

![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/DRC%20Flow.jpg)

Fig. DRC Flow

### Input:
1. Rule File (SVRF / TVF)
2. Layout Database (GDSII, OASIS, Binary, ASCII)
    Binary & ASCII are properietary to Metor Graphics Corporation and only work in flat Calibre nmDRC.

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
    ORIGINAL/ DRAWN LAYER:
    LAYER M1 2                          // simple layer      
    LAYER MET 3 4 5                     // layer set
    
    ![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/drawn%20layers.png)
    
    DERIVED POLYGON LAYER:
    gate = poly and oxide
    
    ![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/derived%20polygon%20layers.png)
    
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
    SVRF statements can refer to layers by name or number.
    A Global layer is defined outside of a rulecheck.
    A Local layer is defined within a rulecheck.
    
2. *Layer Operations*
   Two types - Layer Constructors & Layer Selectors
   ```
    layer1 AND layer2                       // **constructs** /creates new polygons from the polygon data on both input layers.
    layer1 COINCIDENT EDGE layer2           // **selects** edges or edge segments from the first input layer that are coincident with edges from the second input layer.
    ```
3. *Rule Check Statements*
   A rulecheck is added to the rulefile to check one or more design rules.
   They specify layer operations within the rule file that instantiate the resulting derived layers into the DRC (or ERC) results database.
   Syntax:
   
   rule_check_name {
        layer_definition | layer operation
        .
        .
        .
   }
    
    Calibre keeps layer data in memory until it is no longer needed by another rulecheck.
    
    RuleCheck Comments:
    3 ways-     //      OR      @       OR      /* ... */
    The last way of commenting is for multi-line comments.

4. *Dimensional Check Operations*
   The dimensional check operations generate derived error layers, derived edge layers, or derived polygon layers by measuring the separation of edges on one- or two-input layers.
   - Internal
      - For single layer syntax, It measures the separation between internal facing side of edges from same polygon on Layer1. 
      - For two layer syntax, It measures interior facing edges of Layer1 and interior facing edges of Layer2
    
    ![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/Internal%20Check.PNG)
      
   - External
      - For the single-layer syntax, It measures the separations between exterior-facing edges on layer1.
      - For the two-layer syntax, It measures the separations between the exterior-facing sides of layer1 edges and the exterior-facing sides of layer2 edges.
   
    ![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/External%20Check.PNG)
     
   - Coincident Edge
      - Selects all layer1 edges or edge portions coincident with layer2 edges.

    ![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/Coincident%20Edge.PNG)
      
   - Abut
      - Measures the separation between intersecting edges and is interpreted in degrees.
      - Default value is >=0<180 and the output from this operation is in addition to any other output from different operation.
      - Recommended value for any operation in the rule file is Abut<90.

     ![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/Abut%20Operation.PNG)
     
   
   - With Edge
      - Selects all layer1 polygons that share edges or edge segments on layer2.
      
    ![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/With%20Edge%20Operation.PNG)

5. *DRC Output Control Statements*
   - DRC RESULTS DATABASE:
        It specifies filename and type of the results database for CalibrenmDRC.
        > DRC RESULTS DATABASE filename [type][PSEUDO|USER MERGED|USER]

   - DRC MAXIMUM VERTEX:
        It specified maximum vertex count of any polygon DRC result to be written to the DRC Results database.
        > DRC MAXIMUM VERTEX {number | ALL}
     
   - DRC CHECK MAP:
        It controls the database output structure for DRC Rulechecks.
        ```
            DRC CHECK MAP rule_check
            {{GDSII|OASIS}[layer[datatype]]}|ASCII|
            BINARY[filename][MAXIMUM RESULTS{max|ALL}]
            [MAXIMUM VERTICES{maxvertex|ALL}]
            [TEXTTAG name][PSEUDO|USER|USER MERGED]
            {[{AREF cell_name width length
            [minimum_element_count]
            [SUBSTITUTE x1 y1 … xn yn ]}...]|
            [AUTOREF]}
        ```
   - DRC MAP TEXT:
        It specifies whether to translate all text objects in input database to DRC Results Database.
        > DRC MAP TEXT {NO|YES}
     
   - DRC MAP TEXT DEPTH:
        It controls the depth for reading text objects for the DRC MAP TEXT YES specification statements.
        > DRC MAP TEXT DEPTH {ALL|PRIMARY|depth}
     
   - DRC SUMMARY REPORT:
        It specifies DRC Summary Report filename and how it is written.
        > DRC SUMMARY REPORT filename [REPLACE | APPEND][HIER]

   - DRC MAXIMUM RESULTS:
        You can limit the number of DRC results written to the DRC results database for any given DRC rule check by using the DRC Maximum Results specification statement, or by using the MAXIMUM RESULTS parameter to the DRC Check Map specification statement. By default, this maximum result limit is 1000 per DRC rule check.
        > DRC MAXIMUM RESULTS {maxresults | ALL}

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
    LAYER diff 24                   // DIFFUSION
    LAYER poly 5                    // POLY
    LAYER metal2 9                  // METAL2
    LAYER via 12                    // VIA
    
    // ONE OR MORE DERIVED LAYER DEFINITIONS
    gate = poly AND diff            // GATE
    sd = diff NOT gate              // SOURCE-DRAIN
    
    // ONE OR MORE DRC RULECHECKS
    min_gate_length {
    @ Gate length along POLY must be >= 3 microns.
    x = INSIDE EDGE poly diff
    INTERNAL x < 3
    }
    
```
### Output:
1. *Run Transcript* - shows statistics regarding Rule File Compilation, Layout Data Input, Executive Processes. It shows the pathname of the rule file, the contents of the rule file, and the amount of CPU and real time required for compilation, shows cell, layer, and text information, and a summary of the layout data, reports event logs, warning messages, and summary information. It also reports operating parameters, such as maximum results per check, maximum vertices per result polygon (DRC).
    The layer statistics differs for different Calibre applications i.e., the layer statistic for layer1 in flat tun will be different than the layer statistic for layer1 in hierarchical run.
    
2. *DRC Results Database*
    It is a collection of geometric objects grouped together by rule check. DRC Results Database can be ASCII (default), GDSII, OASIS, or Binary. Generally should output an ASCII DRC results database. When you use Calibre nmDRC-H for mask layer generation, you should output a GDSII or OASIS DRC results database, you should also specify DRC Maximum Results ALL in this case. You should follow these guidelines because Calibre nmDRC-H requires extra internal overhead to generate mask layer results.

3. *DRC Summary Report* (optional)
    It includes general information about the run, warnings generated during drc run, list of original layers and number of original shapes processed for that layer, list of rulechecks and number of results generated, total runtime, number of original shapes processed, number of rulechecks executed. If DRC Summary Report specification statement is present only then this file is generated. You can use REPLACE & APPEND keywords to decide whether the file should be overwritten or appended to.

![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/heading_drc_Summary.png)






