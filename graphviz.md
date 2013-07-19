Graphviz & dot basics
=====================

Graphviz website & documentation
--------------------------------

See <http://www.graphviz.org/>

Simple example of creating a directed graph
-------------------------------------------

Nb comments are C-style ie. `//` or `/* ... */`

The basic syntax for defining a directed graph is
[simple_example.dot](graphviz/simple_example.dot):

    // simple_example.dot
    //
    // Simple example of a directed graph
    digraph simple_example {
       step_1 -> step_2;
       step_1 -> step_3;
       step_2 -> step_3;
    }

The graph name is `simple_example`, and the graph nodes (`step_1`, `step_2` and `step_3`)
are defined implicitly in the statements defining the relationships
(e.g. `step_1 -> step_2;`).

To render this example as a PNG using the `dot` layout engine, do:

    % dot -Tpng -o simple_example.png simple_example.dot

`-T` defines the format of the output and `-o` specifies the output file name.


Defining nodes
--------------

Nodes can be defined explicitly outside of the relationship definitions:

    name [attribute=value,...];

for example:

    step_1 [label="Step 1",shape="box"];

Useful attributes:

    label: text that appears in the node (can include "\n" for line breaks)
    shape: shape that the node will be drawn with. Commonly used examples are
           box, ellipse, circle, diamond, plaintext - more esoteric shapes
	   are also available (see appendix H of graphviz manual).

    style:     values are bold, dotted, filled
    color:     node shape colour
    fillcolor: node fill colour

    fontcolor: colour of the label text
    fontname:  name of a font to use for label text
    fontsize:  point size of font for label text

(More attributes are available controlling line widths, text orientation, sizes
etc - see the graphviz documentation.)

Defaults for the node attributes can be explicitly specified within the `digraph`
declaration, e.g.:

    digraph example {
       ...
       node [style=filled,color=white];
       ...

The defaults will take effect for all subsequent nodes (unless over-ridden by
settings for specific nodes, or by a new `node` declaration).


Digraph attributes
------------------

Drawing orientation, size and spacing can be explicitly defined for a directed
graph.

    ranksep: sets the rank separation (distance between rows of nodes) in inches
    ratio: determines the aspect ratio
    orientation: portrait or landscape
    rotate: specify the number of degrees to rotate the image by (rotate=90 is
            equivalent to orientation=landscape).
    page: specify the size of the page that the image should be fitted to
    size: specify the size of the image in inches e.g. size="7.5,10" fits on a
          8.5" by 11" page.


Cluster subgraphs
-----------------

A cluster is a subgraph placed in its own distinct rectangle of the layout.
A subgraph is recognised as a cluster when its name has the prefix cluster, e.g.

    subgraph cluster_example {
        // Define attributes, including style for the subgraph
        node [style=filled,color=white];
	style=filled;
	color=lightgrey;
	
	// Text label for the subgraph
	label = "Process #1";

	// Nodes within the subgraph
	substep1 -> substep_2 -> substep_3;
    }

Subgraph nodes can be referenced from outside the subgraph using their names
without any special syntax.


Constrainted ranks of nodes
---------------------------

Constrainted ranks are a way of controlling the layout of the digraph so that
sets of nodes are placed in line with each other on the same "rank":

    { rank = same; <list of nodes> }

For example:

    { rank = same; step_1; step_2; step_3; }

Alternative values for `rank` are `min`, `source`, `max` and `sink`.

Rank separation can be specified using the global `ranksep` declaration.