The purpose of doing DRC - Design Rule Checks, is to make sure that all the polygons and layers from the layout database follow through the manufacturing process guidelines. This follow through is ensured by running the layout against a DRC rule deck provided by the fabrication team in order for the design to meet functionality, manufacturability and avoid chip failure. If the results thus obtained fall within permissible margin set up by fabs, the design is said to be Clean, else it’s said to be ‘Dirty’ (needs to be fixed and made clean). 

![alt text](https://github.com/divya-gupta-sevya/PDK-/blob/main/DRC%20Flow.jpg)

Fig. DRC Flow

Input:
1. Rule File (SVRF / TVF)
2. Layout Database (GDSII, OASIS, Binary, ASCII)
    Binary & ASCII are properietary to Metor Graphics Corporation and only work in flat Calibre nmDRC.

Output:
1. Run Transcript
2. DRC Results Database
3. DRC Summary Report (optional)

DRC Results Database can be ASCII (default), GDSII, OASIS, or Binary. Generally should output an ASCII DRC results database. 
When you use Calibre nmDRC-H for mask layer generation, you should output a GDSII or OASIS DRC results database, you should also specify DRC Maximum Results ALL
in this case. You should follow these guidelines because Calibre nmDRC-H requires extra internal overhead to generate mask layer results.

Command Line Invocations:

#Flat run (Calibre nmDRC)
calibre -drc rule_file

# Hierarchical run (Calibre nmDRC-H)
calibre -drc -hier rule_file

# Multithreaded hierarchical runs
calibre -drc -hier -turbo -hyper rule_file
calibre -drc -hier -turbo -remote host1,host2,host3 -hyper rule_file



