# Table of Contents
1. [Shredder](#shredder)
2. [Download and Installation](#downloading-and-installing-shredder)  
3. [API](#api)
3. [Usage](#usage)
&nbsp;

Shredder
========
Shredder is a 'c' library to decompose grid graphs into suitable decomposition units.

Classes of graphs considered right now are rectangular grid graphs (RG), rectangular grid graphs with NWSE diagonals (RG_NWSE), rectangular grid graphs with SWNE diagonals (RG_SWNE), rectangular grid graphs with both diagonals (RG_D).

Decomposition units are star graphs and cliques (triangles). Star graphs are 5 point stencil (S4), 7 point stencil (S6), and 9 point stencil (S8).

Each decompostion tries to minimize number of decomposition units, except RG_D to triangles. For decompostion of RG_D to triangles two decompositions (RG_SWNE to triangles and RG_SWNE to triangles) are simiply put together.


Downloading and Installing Shredder
=====================================
To download, and install Shredder run following instructions in terminals.

    cd              		
    git clone https://github.com/neo8git/Shredder.git   #Download Shredder
    ./configure                   # generate make files
    make install       			  # install shredder into system library

To check shredder distribution run followng instructions in terminal. This will also give a distributable zipped copy of shredder. Notice that this will prompt inputs for all test programs descibed below.

	cd              			  # cd into the shredder directory 
    make distcheck       		  # uninstall shredder

To uninstall shredder run followng instructions in terminal.  

	cd              			  # cd into the shredder directory 
    make uninstall       		  # uninstall shredder
   
To clean up files generated by ./configure run followng instruction in terminal.  

	cd              			  # cd into the shredder directory 
    make distclean       		  # uninstall shredder


API
===

After installation, you can use Shredder as an external library by including "shredder.h" in your program, and then compiling with -lshredder flag. API routines with their arguments are as follows.

**status shredder_read_graph(int gtype, int nrows, int ncols, int nnz, int\* rowidx, int\* colptr)**

This function reads a grid graph in CSR format into shredder's memory (will copy the graph into dynamically allocated memory). Caller is reponsible for calling shredder_delete_graph to clear allocated memory. Succesful invocation will return SHREDDER_TOKEN_SUCCESS.

- gtype : can only be one of following SHREDDER_GTYPE_
					SHREDDER_GTYPE_RG (Rectangular grid graphs)
					SHREDDER_GTYPE_RG_NWSE (Rectangular grid graphs with NWSE diagonal)
					SHREDDER_GTYPE_RG_SWSE (Rectangular grid graphs with SWNE diagonal)
					SHREDDER_GTYPE_RG_D (Rectangular grid graphs with both diagonals)
- nrows : number of rows in the grid
- ncols : number of colums in the grid
- nnz :   number of edges in the grid, shredder_create_graph can be used for regular grid,
		  for grid with missing edges this need to be figured out correctly,
		  grid is undirectional, so nnz is twice the # edges
- rowidx : CSR egde pointers
- colptr : CSR edges

**status shredder_stars(int\* size, int \*\* decomposition, int\*\* rowidx, int\*\* colptr)**

Decomposes the graph loaded into Shredder's memory into stars, and saves into the arguments
passed. The graph in Shredder's memory will be kept intact, the caller is responsible for
freeing the memory used for storing the decompositions, once the need is over. This can be
conveniently done using shredder_delete_stars.

- size : Address of the variable where size of the decomposition will be stored.
- decomposition : Address of a pointer where an array of decompositions will be stored. 
				  The array will contain indices of the rowidx corresponding 
				  to the vertices picked
- rowidx : similar to CSR format consecutive entries will hold the range of indices into 
		   colptr holding all the edges of a star, 
		   caller will need to pass address of a pointer
- colptr : CSR edges holding only the edges of the stars picked as decomposition, 
		   caller will need to pass address of a pointer


**status shredder_cliques(int\* size, int\*\*\* decomposition)**

Decomposes the graph loaded into Shredder's memory into cliques (right now only triangles), 
and saves in the arguments passed, the graph in Shredder's memory will be keet intact, 
the caller is reposible for freeing the memory used for storing the decompositions, once 
the need is over. This can be conveniently done using shredder_delete_cliques.

- size : Address of the variable where size of the decomposition will be stored.
- decomposition : Address of a double pointer where a 2D array of decompositions will be stored.
				  The array will contain indices of the rowidx corresponding the vertices picked from graph loaded through shredder_read_graph.
				  Each row in the array will contain a clique.
 

**double shredder_decomposition_time()**

Will return the time spend for decomposing a graph already loaded into Shredder's memoroy. 
This function can be used either for star or clique decomposition. The function should be
called after the call of shredder_stars or shredder_cliques. Timing information will
be lost once shredder_delete_graph is called. Unit of time is second.

**status shredder_delete_graph()**

Clears shredder's memory, shredder can be reused for loading a new graph after this operation
 
**status shredder_delete_stars(int\* size, int\*\* decomposition, int\*\* rowidx, int\*\* colptr)**

Clears memory allocated for stars through shreder_stars.

- size : Address of the variable where size of the decomposition is stored
- decomposition : Address of a pointer where an array of decompositions 
	 			  is stored.
- rowidx : caller will need to pass address of a pointer
- colptr : caller will need to pass address of a pointer							


**status shredder_delete_cliques(int\* size, int\*\*\* decomposition)**

Clears memory allocated for cliques through shreder_cliques

- size : Address of the variable where size of the decomposition is stored.
- decomposition : Address of a double pointer where a 2D array of decompositions 
	 			  is stored.

**status shredder_create_graph(int gtype, int nrows, int ncols, int\* nnz, int\*\* rowidx, int\*\* colptr)**
	
Creates a regular grid graph in CSR format, which can be passed to shredder_read_graph.
This is a conveient routine for creating a regular grid graph of graph type defined 
above (GTYPE_...). If the grid graph has missing edges, then this should not be used. 

- gtype : can only be one of SHREDDER_GTYPE_ (e.g. SHREDDER_GTYPE_RG_NWSE)
- nrows : number of rows in the grid
- ncols : number of colums in the grid
- nnz :   Address of an integer variable to store number of edges in the grid
- rowidx : Addres of a pointer to store CSR egde pointers
- colptr : Addres of a pointer to store CSR edges

**status shredder_print_graph()**

Prints graph loaded into shredder's memory

**status shredder_print_stars(int size, int\* decomposition, int\* rowidx, int\* colptr)**

Prints all the stars and associated edges of the stars
 
**status shredder_print_cliques(int size, int\*\* decomposition)**
	
Prints all the cliques of the decomposition array, right now only prints traingles for each decomposition
 

Usage
=====

Several test prgorams are provided in test directory. These can be compiled and run using shredder library as follows.

	cd ../test
	gcc -o exec test_program.c -lshredder
	./exec

The test programs provided are as follows. 

All test programs prompt for 3 inputs except the last one: < graph_type > <#rows> <#columns>. 

	graph_type: one of RG_GTYPE_, have to be between [0-3] 
	#rows: number of rows (>1)
	#columns: number of columns (>1)


**create.c**

This is a test program to check the routine of creating regualr grid graphs in CSR format (shredder_create_graph).

Successful invocation on shredder_create_graph will return SHREDDER_TOKEN_SUCCESS, and load the CSR into argument passed, the CSR can then be passed to other Shredder routines. Notice that shredder_create_graph can only create regular graphs.

**read.c**

This is test program to test loading a CSR graph into Shredder's memory (shredder_read_graph), as well as cleaning Shredder's memory (shredder_delete_graph). 

Successful invocation of shredder's routine will return SHREDDER_TOKEN_SUCCESS, and load the CSR into Shredder's memory. After that any suitable decomposition routine can be invoked to get desired decomposition.

This routine should be carefully coupled with call to shredder_delete_graph, otherwise there will be memory leaks. If this is the only call to shredder_read_graph then there should be single call to shredder_delete_graph at the end of using Shredder. Otherwise every successive call to shredder_read_graph should be precedded by a call to shredder_delete_graph.

There is a routine (shredder_print_graph) in the library to print the CSR graph loaded into Shredder.

**stars.c**

This is test program to grab a star decomposition of a graph loaded into Shredder's memory (using shredder_read_graph), as well as to get corresponding decomposition time. The decomposition will be loaded into the argument passed, and caller is reposible for freeing the memory associated with decomposition.

This program also uses a library rouinte shredder_delete_stars to clean the memory used to store decomposition.

**cliques.c**

This is test program to grab a clique decomposition of a graph loaded into Shredder's memory (using shredder_read_graph), as well as to get corresponding decomposition time. The decomposition will be loaded into the argument passed, and caller is reposible for freeing the memory associated with decomposition.

Notice that SHREDDER_GTYPE_RG does not support clique decomposition.

This program also uses a library rouinte shredder_delete_cliques to clean the memory used to store decomposition.

**example.c**
This is test program which usages multiple routines of Shredder at once. 

This test program prompts for 4 inputs: < graph_type > < decompostion_type > <#rows> <#columns>. 

	graph_type: one of SHREDDER_GTYPE_, [0-3] 
	decomposition_type: either SHREDDER_DTYPE_STAR or SHREDDER_DTYPE_CLIQUE
	#rows: number of rows (>1)
	#columns: number of columns (>1)

On a valid set of input it will print star or clique decomposition time of a graph passed as an input. If anything goes wrong the program will terminate with EXIT_FAILURE status.

&nbsp;  
&nbsp;  
&nbsp;
