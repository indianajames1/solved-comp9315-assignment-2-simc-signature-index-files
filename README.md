Download Link: https://assignmentchef.com/product/solved-comp9315-assignment-2-simc-signature-index-files
<br>



This assignment aims to give you an understanding of

how database files are structured and accessed

how <strong>s</strong>uper<strong>im</strong>posed <strong>c</strong>odeword (SIMC) signatures are implemented how partial-match retrieval searching is implemented using SIMC signatures

The goal is to build a simple implementation of a SIMC signature file, including applications to create SIMC files, insert tuples into them, and search for tuples based on partial-match retrieval queries.

<strong>Summary</strong>

<table width="612">

 <tbody>

  <tr>

   <td width="89"><strong>Deadline</strong>:</td>

   <td width="523">23:59pm on <strong>Sunday 26 April</strong></td>

  </tr>

  <tr>

   <td width="89"><strong>Late Penalty</strong>:</td>

   <td width="523">0.125 <em>marks</em> off the ceiling mark for each hour late (i.e. 3 marks/day)</td>

  </tr>

  <tr>

   <td width="89"><strong>Marks:</strong></td>

   <td width="523">contributes <strong>15 marks</strong> toward your total mark for this course.(note that the item marks sum to 18; this will be converted into a mark out of 15 by multiplying by 15/18)</td>

  </tr>

  <tr>

   <td width="89"><strong>Groups:</strong></td>

   <td width="523">do this assignment in pairs or individually (you can use the same groups as for Assignment 1)</td>

  </tr>

  <tr>

   <td width="89"><strong>Submission</strong>:</td>

   <td width="523">login to Course Web Site &gt; Assignments &gt; Assignment 2 &gt; Submission &gt; upload  ass2.tar or login to any CSE server &gt; give cs9315 ass2 ass2.tar</td>

  </tr>

  <tr>

   <td width="89"><strong>Workspace:</strong></td>

   <td width="523">any machine wth a C compiler (preferably gcc); you do not need to use Grieg</td>

  </tr>

 </tbody>

</table>

The ass2.tar file must contain the Makefile plus all of the *.c and *.h files that are needed to compile the create, insert and select executables.

You are <em>not</em> allowed to change the following files: create.c, insert.c, select.c, stats.c, dump.c, hash.h, hash.c, x1.c, x2.c, x3.c. We supply them when we/you test your files, so any changes you make will be overwritten. Do not include them in the ass2.tar file. Details on how to build the ass2.tar file are given below.

Note that the create.c, insert.c, select.c, stats.c, dump.c code assumes that you honour the interfaces to the ADTs defined in the *.[ch] file pairs. If you change the interfaces to data types like bits.h and page.h, then your program will be treated as incorrect.

Make sure that you read this assignment specification <em>carefully and completely</em> before starting work on the assignment. Questions which indicate that you haven’t done this will simply get the response “Please read the spec”.

<strong>Note:</strong> this assignment does not require you to do anything with PostgreSQL.

<strong>Introduction</strong>

Signatures are a style of indexing where (in its simplest form) each tuple is associated with a compact representation of its values (i.e. its <em>signature</em>). Signatures are used in the context of partial-match retrieval queries, and are particularly effective for large tuples. Selection is performed by first forming a <em>query signature</em>, based on the values of the known attributes, and then scanning the stored signatures, matching them against the query signature, to identify potentially matching tuples. Only these tuples are read from the data pages and compared against the query to check whether they are true matching tuples. Signature matching allows for “false matches”, where the query and tuple signatures match, but the tuple is not a valid result for the query. Note that signature matching can be quite efficient if the signatures are small, and efficient bit-wise operations are used to check for signature matches.

The kind of signature matching described above uses one signature for each tuple (as in the diagram below). Other kinds of signatures exist, and one goal is to implement them and compare their performance to that of tuple signatures.

In files such as the above, queries are evaluated as follows:

<table width="656">

 <tbody>

  <tr>

   <td width="656">Input: pmr query, Output: set of tuples satisfying the query qrySig = makeSignature(query)Pages = {}  // set of pages containing possibly matching tuples foreach tupSig in SignatureFile {     if (tupSig matches qrySig) {// potential matchPID = page of tuple associated with tupSig        add PID to Pages}}Results = {}  // set of tuples satisfying queryforeach PID in Pages {     buf = fetch data page PID     foreach tuple in buf {         // check for real matchif (tuple satisfies query) add tuple to Results}}</td>

  </tr>

 </tbody>

</table>

Note that above algorithm is an abstract view of what you must implement in your code. The function makeSignature() does not literally exist, but you need to build analogues to it in your code.

Signatures can be formed in several ways, but we will consider only signatures that are formed by <strong>s</strong>uper<strong>im</strong>posing <strong>c</strong>odewords (SIMC). Each codeword is formed using the value from one attribute.

In our system, a SIMC relation R is represented by five physical files:

R.info containing global information such as

the number of attributes and size of each tuple the number of data pages and number of tuples the sizes of the various kinds of signatures the number of signatures and signature pages etc. etc. etc.

The R.info file contains a copy of the RelnParams structure given in the reln.h file (see below).

R.data containing data pages, where each data page contains

a count of the number of tuples in the page the tuples (as comma-separated character sequences)

Each data page has a capacity of <em>c</em> tuples. If there are <em>n</em> tuples then there will be <em>b</em> = ⌈<em>n/c</em>⌉ pages in the data file. All pages except the last are full. Tuples are never deleted.

R.tsig containing tuple signatures, where each page contains

a count of the number of signatures in the page the signatures themselves (as bit strings)

Each tuple signature is formed by superimposing the codewords from each attribute in the tuple. If there are <em>n</em> tuples in the relation, there will be <em>n</em> tuple signatures, in <em>b</em><em><sub>t</sub></em> pages. All tuple signature pages except the last are full.

R.psig containing page signatures, where each page contains

a count of the number of signatures in the page the signatures themselves (as bit strings)

Page signatures are much larger than tuple signatures, and are formed by superimposing the codewords of all attribute values in all tuples in the page. There is one page signature for each page in the data file.

R.bsig containing bit-sliced signatures, where each page contains

a count of the number of signatures in the page the bit-slices themselves (as bit strings)

Bit-slices give an alternate 90<sup>o</sup>-rotated view of page signatures. If there are <em>b</em> data pages, then each bit-slice is <em>b</em>-bits long. If page signatures are <em>pm</em> bits long, then there are <em>pm</em> bit-slices.

The following diagram gives a very simple example of the correspondence between page signatures and bit-slices:

The different types of pages (tuple, signature, slice) were described above. Internally, all pages have a similar structure: a counter holding the number of items in the page, and the items themselves (tuples or signatures or slices). All of the items in a page are the same size. The following diagram shows the structure of pages in the files of a SIMC relation:

We have developed some infrastructure for you to use in implementing SIMC files. The code we give you is not complete; you can find the bits that need to be completed by searching for TODO in the code.

How you implement the missing parts of the code is up to you, but your implementation must conform to the conventions used in our code. In particular, you should preserve the interfaces to the supplied modules (e.g. Bits, Reln, Query, Tuple) and ensure that your submitted modules work with the supplied code in the create, insert and select commands.

<strong>Commands</strong>

In our context, SIMC-indexed relations are a collection of files that represent one relational table. These relations can be manipulated by a number of supplied commands:

<strong>create</strong> RelName #tuples #attrs 1/pF

Creates an empty relation called <em>RelName</em> with all tuples having <em>#attrs</em> attributes. The <em>#tuples</em> parameter gives the expected number of tuples that are likely to be inserted into a relation; this, in turn, determines parameters like the number of data pages and length of bit-sliced superimposed codewords. The <em>1/pF</em> parameter gives the inverse of the false match probability; for example, a value of 1000 gives a false match probability of 1/1000 (0.001).

These parameters are combined using the formulas given in lectures to determine how large tuple- and page-signatures are. Each bit-slice has a number of bits equal to the number of data pages, which is determined from <em>#attrs</em>, <em>#tuples</em> and the page size.

This gives you storage for one relation/table, and is analogous to making an SQL data definition like:

create table R ( a<sub>1</sub> integer, a<sub>2</sub> text, … a<sub>n</sub> text );

Note that internally, attributes are indexed 0..<em>n</em>-1 rather than 1..<em>n</em>.

The following example of using create makes a relation called abc where each tuple has 4 attributes and the indexing has a false match probability of 1/100. The relation can hold up to 10000 tuples (it can actually hold more, but only the first 10000 will be indexed via the bit-sliced signatures).

$ <strong>./create  abc  10000  4  100</strong>

<strong>insert</strong> RelName

Reads tuples, one per line, from standard input and inserts them into the relation specified on the command line. Tuples all take the form <em>val</em><em><sub>1</sub>,val</em><em><sub>2</sub>,…,val</em><em><sub>n</sub></em>. The values can be any sequence of alpha-numeric characters and ‘-‘. The characters ‘,’ (field separator) and ‘?’ (query wildcard) are treated specially.

Since all tuples need to be the same length, it is simplest to use gendata to generate them, and pipe the generated tuples into the insert command

<strong>select</strong> RelName QueryString SigType

Takes a “query tuple” on the command line, and finds all tuples in the data pages of the relation <em>RelName</em> that match the query. Queries take the form <em>val</em><em><sub>1</sub>,val</em><em><sub>2</sub>,…,val</em><em><sub>n</sub></em>, where some of the <em>val</em><em><sub>i</sub></em> can be ‘?’ (without the quotes). Some examples, and their interpretation are given below. You can find more examples in the lecture slides and course notes.

?,?,?    # matches any tuple in the relation

10,?,?   # matches any tuple with 10 as the value of attribute 1

?,abc,?  # matches any tuple with abc as the value of attribute 2

10,abc,? # matches any tuple with 10 and abc as the values of attributes 1 and 2

There are also a number of auxiliary commands to assist with building and examining relations:

<strong>gendata</strong> #tuples #attributes [startID] [seed]

Generates a specified number of <em>n</em>-attribute tuples in the appropriate format to insert into a created relation. All tuples are the same format and look like

<em>UniqID</em>,<em>RandomString</em>,a3-<em>Num</em>,a4-<em>Num</em>,…,a<sub>n</sub>–<em>Num</em>

For example, the following 4-attribute tuples could be generated by a call like   gendata 1000 4

7654321,aTwentyCharLongStrng,a3-013,a4-001

3456789,aTwentyChrLongString,a3-042,a4-128

Of course, the above call to gendata will generate 1000 tuples like these.

A tuple is represented by a sequence of comma-separated fields. The first field is a unique 7-digit number; the second field is a random 20-char string (most likely unique in a given database); the remaining fields have a field identifier followed by a nonunique 3-digit number. The size of each tuple is

7+1 + 20+1 + (<em>n</em>-2)*(6+1)-1  = 28 + 7*(n-2) bytes

The -1 is because the last attribute doesn’t have a trailing comma, and (<em>n</em>-2)*(6+1) assumes that it does.

Note that tuples are limited to at most 9 attributes, which means that the maximum tuple size is a modest 77 bytes. (If you wish, you can work with larger tuples by tweaking the gendata and create commands and the newRelation() function, but this not required for the assignment).

<strong>stats</strong> RelName

Prints information about the sizes of various aspects of the relation. Note that some aspects are static (e.g. the size of tuples) and some aspects are dynamic (e.g. the number of tuples). An example of using the stats command is given below.

You can use it to help with debugging, by making sure that the files have been correctly built after the create command, and that the files have been correctly updated after some tuples have been inserted.

<strong>dump</strong> RelName

Writes all tuples from the relation <em>RelName</em>, one per line, to standard output. This is like an inverse of the insert command. Tuples are dumped in a form that could be used by insert to rebuild a database.

You can use it to help with debugging, by making sure that the tuples are inserted correctly into the data file.

<strong>Setting Up</strong>

You should make a working directory for this assignment and put the supplied code there, and start reading to make sure that you understand all of the data types and operations used in the system.

$ <strong>mkdir <em>your/ass2/directory</em></strong>

$ <strong>cd <em>your/ass2/directory</em></strong>

$ <strong>unzip /web/cs9315/20T1/assignments/ass2/ass2.zip</strong>

You should see the following files in the directory:

<table width="656">

 <tbody>

  <tr>

   <td width="118">$ <strong>ls</strong>Makefile</td>

   <td width="111">dump.c</td>

   <td width="111">psig.c</td>

   <td width="111">stats.c</td>

   <td width="204">x1.c</td>

  </tr>

  <tr>

   <td width="118">bits.c</td>

   <td width="111">gendata.c</td>

   <td width="111">psig.h</td>

   <td width="111">tsig.c</td>

   <td width="204">x2.c</td>

  </tr>

  <tr>

   <td width="118">bits.h</td>

   <td width="111">hash.c</td>

   <td width="111">query.c</td>

   <td width="111">tsig.h</td>

   <td width="204">x3.c</td>

  </tr>

  <tr>

   <td width="118">bsig.c</td>

   <td width="111">hash.h</td>

   <td width="111">query.h</td>

   <td width="111">tuple.c</td>

   <td width="204"> </td>

  </tr>

  <tr>

   <td width="118">bsig.h</td>

   <td width="111">insert.c</td>

   <td width="111">reln.c</td>

   <td width="111">tuple.h</td>

   <td width="204"> </td>

  </tr>

  <tr>

   <td width="118">create.c</td>

   <td width="111">page.c</td>

   <td width="111">reln.h</td>

   <td width="111">util.c</td>

   <td width="204"> </td>

  </tr>

  <tr>

   <td width="118">defs.h</td>

   <td width="111">page.h</td>

   <td width="111">select.c</td>

   <td width="111">util.h</td>

   <td width="204"> </td>

  </tr>

 </tbody>

</table>

The .h files define data types and function interfaces for the various types used in the system. The corresponding .c files contain the implementation of the functions on the data type. The remaining .c files either provide the commands described above, or are test harnesses for individual types (x1.c, x2.c, x3.c). You can add additional testing files, bu there is no need to submit them.

The above files give you a partial implementation of SIMC indexing. You need to complete the code so that it provides the functionality described above.

You should be able to build the supplied partial implementation via the following:

<table width="656">

 <tbody>

  <tr>

   <td width="656">$ <strong>make</strong>gcc -std=gnu99 -Wall -Werror -g   -c -o query.o query.c gcc -std=gnu99 -Wall -Werror -g   -c -o page.o page.c gcc -std=gnu99 -Wall -Werror -g   -c -o reln.o reln.c gcc -std=gnu99 -Wall -Werror -g   -c -o tuple.o tuple.c gcc -std=gnu99 -Wall -Werror -g   -c -o util.o util.c gcc -std=gnu99 -Wall -Werror -g   -c -o tsig.o tsig.c gcc -std=gnu99 -Wall -Werror -g   -c -o psig.o psig.c gcc -std=gnu99 -Wall -Werror -g   -c -o bsig.o bsig.c gcc -std=gnu99 -Wall -Werror -g   -c -o hash.o hash.c gcc -std=gnu99 -Wall -Werror -g   -c -o bits.o bits.c gcc -std=gnu99 -Wall -Werror -g   -c -o create.o create.cgcc -o create create.o query.o page.o reln.o tuple.o util.o tsig.o psig.o bsig.o hash.o bits.o -lmgcc -std=gnu99 -Wall -Werror -g   -c -o insert.o insert.cgcc   insert.o query.o page.o reln.o tuple.o util.o tsig.o psig.o bsig.o hash.o bits.o   -o insertgcc -std=gnu99 -Wall -Werror -g   -c -o select.o select.cgcc   select.o query.o page.o reln.o tuple.o util.o tsig.o psig.o bsig.o hash.o bits.o   -o select gcc -std=gnu99 -Wall -Werror -g   -c -o stats.o stats.cgcc   stats.o query.o page.o reln.o tuple.o util.o tsig.o psig.o bsig.o hash.o bits.o   -o statsgcc -std=gnu99 -Wall -Werror -g   -c -o gendata.o gendata.cgcc -o gendata gendata.o util.o -lmgcc -std=gnu99 -Wall -Werror -g   -c -o dump.o dump.cgcc   dump.o query.o page.o reln.o tuple.o util.o tsig.o psig.o bsig.o hash.o bits.o   -o dump gcc -std=gnu99 -Wall -Werror -g   -c -o x1.o x1.cgcc -o x1 x1.o query.o page.o reln.o tuple.o util.o tsig.o psig.o bsig.o hash.o bits.o gcc -std=gnu99 -Wall -Werror -g   -c -o x2.o x2.cgcc -o x2 x2.o query.o page.o reln.o tuple.o util.o tsig.o psig.o bsig.o hash.o bits.o gcc -std=gnu99 -Wall -Werror -g   -c -o x3.o x3.cgcc -o x3 x3.o query.o page.o reln.o tuple.o util.o tsig.o psig.o bsig.o hash.o bits.o</td>

  </tr>

 </tbody>

</table>

This should not produce any errors on the CSE servers; let me know ASAP if this is not the case.

The gendata command should work completely without change. For example, the following command generates 5 tuples, each of which has 4 attributes. Values in the first attribute are unique; values in the second attribute are highly likely to be unique. Note that the third and fourth attributes cycle through values at different rates, so they won’t always have the same number.

$ <strong>./gendata 5 4</strong>

1000000,lrfkQyuQFjKXyQVNRTyS,a3-000,a4-000 -&gt; 0

1000001,FrzrmzlYGFvEulQfpDBH,a3-001,a4-001 -&gt; 0

1000002,lqDqrrCRwDnXeuOQqekl,a3-002,a4-002 -&gt; 0

1000003,AITGDPHCSPIjtHbsFyfv,a3-003,a4-003 -&gt; 0

1000004,lADzPBfudkKlrwqAOzMi,a3-004,a4-004 -&gt; 0

The create command itself is complete, but some of the functions it calls are not complete. It will allow you to make an empty relation, although without a complete bit-slice file (you add this as one of the assignment tasks). The stats command is complete and can display information about a relation. Using these commands, you could do the following: use the create command to create an empty relation which can hold 4-attribute tuples and able to index up to 5000 tuples (using bit-slices), with a false match probability of 1/1000. The stats command then displays the generated parameter values.

<table width="656">

 <tbody>

  <tr>

   <td width="656">$ <strong>./create R 5000 4 1000</strong>$ <strong>./stats R</strong> Global Info: Dynamic:#items:  tuples: 0  tsigs: 0  psigs: 0  bsigs: 0   #pages:  tuples: 1  tsigs: 1  psigs: 1  bsigs: 1Static:   tups   #attrs: 4  size: 42 bytes  max/page: 97   sigs   bits/attr: 9tsigs  size: 64 bits (8 bytes)  max/page: 511   psigs  size: 5584 bits (698 bytes)  max/page: 5   bsigs  size: 56 bits (7 bytes)  max/page: 584 $</td>

  </tr>

 </tbody>

</table>

You can apply the formulae for calculating the various quantities to check that the above values make sense. Note that the bits for signatures are rounded up to the next multiple of 8 (why waste a few bits?). Note also that all pages are defined to be 4096 bytes. Finally, note that create makes a file with one empty page for each of the files holding tuples and signatures.

As supplied, the insert command inserts tuples into the data pages, but does not generate any signatures. Using gendata is the easiest (and safest) way to add valid tuples. You can then check that the tuples have been inserted via the dump command, and see how the parameters have changed using stats again.

<table width="656">

 <tbody>

  <tr>

   <td width="656">$ <strong>./gendata 5 4 | ./insert -v R</strong>Inserting: 1000000,lrfkQyuQFjKXyQVNRTyS,a3-000,a4-000Inserting: 1000001,FrzrmzlYGFvEulQfpDBH,a3-001,a4-001Inserting: 1000002,lqDqrrCRwDnXeuOQqekl,a3-002,a4-002Inserting: 1000003,AITGDPHCSPIjtHbsFyfv,a3-003,a4-003Inserting: 1000004,lADzPBfudkKlrwqAOzMi,a3-004,a4-004$ <strong>./stats R</strong> Global Info: Dynamic:#items:  tuples: 5  tsigs: 0  psigs: 0  bsigs: 0#pages:  tuples: 1  tsigs: 1  psigs: 1  bsigs: 1Static:tups   #attrs: 4  size: 42 bytes  max/page: 97   sigs   bits/attr: 9tsigs  size: 64 bits (8 bytes)  max/page: 511   psigs  size: 5584 bits (698 bytes)  max/page: 5   bsigs  size: 56 bits (7 bytes)  max/page: 584 $</td>

  </tr>

 </tbody>

</table>

Note that the only difference between the above stats and the stats for the newly-created file is the 5 tuples. There are no signatures, no new pages, etc.

The dump command is complete; it simply scans the data file and displays any tuples it finds, e.g.

$ <strong>./dump R</strong>

1000000,lrfkQyuQFjKXyQVNRTyS,a3-000,a4-000

1000001,FrzrmzlYGFvEulQfpDBH,a3-001,a4-001

1000002,lqDqrrCRwDnXeuOQqekl,a3-002,a4-002

1000003,AITGDPHCSPIjtHbsFyfv,a3-003,a4-003

1000004,lADzPBfudkKlrwqAOzMi,a3-004,a4-004 $

The select command, as supplied, is not complete. However, once it is working (at least with tuple signatures), you should be able to ask queries like:

$ <strong>./select  R  ‘1000001,?,?’  t</strong>       # not enough attrs Invalid query: 101,?,?

$ <strong>./select  R  ‘1000001,?,?,?’  t</strong>

1000001,FrzrmzlYGFvEulQfpDBH,a3-001,a4-001

Query Stats:

# signatures read:   5

<a href="#_Toc23656"># sig pages read:                                                                               1 </a>

<a href="#_Toc23657"># tuples examined:   5                                                                           </a>

<a href="#_Toc23658"># data pages read:                                                                              1 </a>

<a href="#_Toc23659">$ <strong>./select  R  ‘1000001,?,a3-002,?’  t</strong>                                                           </a>

<a href="#_Toc23660">Query Stats: # signatures read:                                                                 5 </a>




# false match pages: 0

<h1><a name="_Toc23656"></a># sig pages read:    1</h1>

<h1><a name="_Toc23657"></a># tuples examined:   0</h1>

<h1><a name="_Toc23658"></a># data pages read:   0</h1>

# false match pages: 0

<h1><a name="_Toc23659"></a>$ <strong>./select  R  ‘1000001,?,a3-002,?’  </strong><strong>x</strong></h1>

<h1><a name="_Toc23660"></a>Query Stats:</h1>

# signatures read:   0

# sig pages read:    0

# tuples examined:   5

# data pages read:   1

# false match pages: 1

Some explanation:

The second query finds a match because there is a tuple with the value 1000001 for its first attribute. The ? represent “don’t care” or wild-card values.

The third query fails because the tuple with 1000001 for its first attribute, does not have the value a3-002 for its third attribute.

The fourth query performs a linear scan of the data file. Since the query itself is the same as the third query, there are no matching tuples. This query reads every data page (there is only one). Any data page read, which does not contain matching tuples, is counted as a “false page match”.

The t at the end of the query tells the query evaluator to use tuple signatures as a first-pass filter. Other possibilities are p for page signatures or b for bit-sliced signatures. If you use a character other than t, p, or b, or don’t specify a signature type, the evaluator uses a linear scan and checks all tuples.

With all types of signatures, queries run in two phases:

use the signatures to determine which pages may contain matching tuples read each of these pages and check all tuples to find real matches

The query statistics are maintained in a Query data structure while the query is executing.

<strong>Data Types</strong>

There are four important data types defined in the system:

<strong>Relations</strong> (data type Reln)

Relations are defined by three data types: Reln, RelnRep, RelnParams. Reln is just a pointer to a RelnRep object; this is useful for passing to functions that need to modify some aspect of the relation structure. RelnRep is a representation of an open relation and contains the parameters, plus file descriptors for all of the open files. RelnParams is a list of various properties of the database. See reln.h for details.

<strong>Queries</strong> (data type Query)

Queries are defined via a QueryRep structure which contains fields to represent the current state of the scan for the query, plus a collection of statistics counters. It is essentially like the query iteration structures described in lectures, and is used to control and monitor the query evaluation. The QueryRep structure also contains a reference to the relation being queried, and a copy of the query string. The Query data type is simply a pointer to a QueryRep structure. See query.h for details. The following diagram might also help:

<strong>Pages</strong> (data type Page)

Pages are defined via a PageRep structure which contains a counter for the number of items, and then an array of bytes containing the actual items, whether they are tuples or signatures or slices. The size of each type of item is held in the RelnParams structure, and so Pages are typically considered in conjunction with Relns. The Page data type is simply a pointer to a PageRep structure. See page.h for details. The following diagram might also help:

<strong>Bit-strings</strong> (data type Bits)

Bit-strings are defined via a BitsRep structure which contains two counters (one for the number of bits, and the other for the number of bytes used to represent the bit-string). The BitsRep structure also contains an array of bytes which hold the bits in the string; the array is created when and instance of a Bits data type is created. Note that Bits is an ADT, so the concrete data structure is hidden from its clients; the Bits data type is simply a pointer to a BitsRep structure. See bits.c for details of the data structure, and bits.h for the function interface. The following diagram might help:

<strong>Tuple</strong> (data type Tuple)

Tuples are just character sequences (like C strings). See tuple.h for details.

There are also a range of (hopefully) self-explanatory data types defined in defs.h. The various signature types are represented as bit-strings (Bits).

<strong>Goal</strong>

Your goal for this assignment is to complete the implementation of the various components of the system, so that it can handle all three kinds of signatures. This includes adding/modifying signatures when new tuples are added, and using these signatures in answering queries. Note that ew don’t consider operations like DELETE or UPDATE in this assignment.

The header (.h) files contain definitions of the data types used in the system.

Each of the source code (.c)files contains comments on each function, describing briefly what it should do. Some functions contain TODO comments to indicate where you need to complete them. You can put all of the code in the indicated function, or you can write new functions that these functions use.

You are free to change any file except create.c, insert.c, select.c and hash.c. Since you can’t change these files, you also cannot change the interfaces to the data types that they use (Reln, Query, Page, Bits). Basically, all of the functions mentioned in the .h files for these types must exist, with the same interface, but you can implement their internals however you like. You can also add extra functions to each data type (i.e. extend its interface) if that helps.

The files x1.c, x2.c, x3.c can be changed, but aren’t relevant to the assignment, except to help with debugging some of the data types.

<strong>Task 1: A Bit-string Type </strong>(2<sub> marks)</sub>

Implement all of the incomplete functions in the bits.c file, to produce a working bit-string data type. The functions to complete are flagged with TODO, and the purpose of each should be clear from the comment at the start of the function and its name. The x1.c file contains some simple test cases for the Bits type.

<strong>Task 2: Scanning for Results </strong>(3<sub> marks)</sub>

After you have Bits working, you can start to implement query evaluation, although without indexing. The startQuery() function parses the query string and then uses the appropriate type of signature to generate a list of pages which potentially contain matching tuples. This list is implemented as a bit-string where a 1 indicates a page which needs to be checked for matches. At this stage, all of the signature types mark all pages as potential matches, so all pages need to be checked.

Note that the startQuery() function can return NULL. It should do so only if the query string contains the wrong number of attributes for the relation.

Before this will work, you need to implement the scanAndDisplayMatchingTuples() function, which performs the check for matching tuples in each of the marked pages. This function, as well as finding and displaying result tuples, maintains the query statistics for number of data pages read, and number of pages that were read but contained no matching tuples.

For this task, you need to complete the scanAndDisplayMatchingTuples() function from the query.c file. This function behaves roughly as follows:

<table width="656">

 <tbody>

  <tr>

   <td width="656">foreach PID in 0 .. npages-1 {     if (PID is not set in MatchingPages)         ignore this page     for each tuple T in page PID {         if (T matches the query string)             display it as a query result}if (no tuples in page PID are results)         count it as a false match page }</td>

  </tr>

 </tbody>

</table>

<strong>Task 3: Tuple Signatures </strong>(4<sub> marks)</sub>

Implement indexing by using tuple-based signatures (i.e. each tuple has its own signature, stored in the <em>Rel</em>.tsig file). You will need to complete the makeTupleSig() and findPagesUsingTupSigs() functions in the tsig.c file, and add some code to the addToRelation() function in reln.c.

The addToRelation() function inserts a tuple into the next available slot in the data file, but currently does nothing with signatures. You should add code here which generates a tuple signature for the new tuple and inserts it in the next available slot in the <em>Rel</em>.tsig file.

The makeTupleSig() function takes a tuple and returns a bit-string which contains a superimposed codeword signature for that tuple. It behaves roughly as follows:

Tsig = AllZeroBits

for each attribute A in tuple T {

CW = codeword for A

Tsig = Tsig OR CW

}

A method for computing codewords is given in the lecture notes.

The findPagesUsingTupSigs() take a tuple signature and scans the <em>Rel</em>.tsig file, comparing that signature to the stored tuple signatures. It builds a bit-string showing which pages contain at least one “matching” tuple. It behaves roughly as follows:

<table width="656">

 <tbody>

  <tr>

   <td width="656">QuerySig = makeTupleSig(Query) Pages = AllZeroBits foreach Tsig in tsigFile {     if (Tsig matches QuerySig) {PID = data page for tuple corresponding to Tsig         include PID in Pages}}</td>

  </tr>

 </tbody>

</table>

Note that the i<sup>th</sup> tuple in the data file has its correpsonding signature as the i<sup>th</sup> signature in the <em>Rel</em>.tsig file. However, since tuples and tuple signatures are different sizes, the page that the signature appears on will not necessarily have the same page ID as the page in which the corresponding tuple is located.

<strong>Task 4: Page Signatures </strong>(4<sub> marks)</sub>

Implement indexing using page-level signatures (psigs).

This is similar to how tuple-level signature indexing is done, except that the sigantures are larger. The functions that you need to complete are makePageSig() and findPagesUsingPageSigs() in the psig.c file. You will also need to add more code to the addToRelation() function to maintain page signatures when new tuples are inserted.

One major difference between tuple signatures and page signatures is that page signatures are not a one-off insertion. When a new tuple is added, its page-level signature needs to be included page signature for the page where it it is inserted. The process can be described roguhly as follows:

new Tuple is inserted into page PID

Psig = makePageSig(Tuple)

PPsig = fetch page signature for data page PID from psigFile

merge Psig and PPsig giving a new PPsig update page signature for data page PID in psigFile

The makePageSig() function be used to generate a page-level signature for the query, and then used to generate a bit-string of matching pages roughly as follows:

<table width="656">

 <tbody>

  <tr>

   <td width="656">QuerySig = makePageSig(Query) Pages = AllZeroBits foreach Psig in psigFile {     if (Psig matches QuerySig) {PID = data page corresponding to Psig         include PID in Pages</td>

  </tr>

 </tbody>

</table>

}

}

<strong>Task 5: Bit-sliced Signatures </strong>(5<sub> marks)</sub>

Implement indexing using bit-sliced page signatures.

Each bit-slice is effectively a list of pages that have a specific bit from the page-signature set to 1 (e.g. if a page-level signature has bit 5 set to one, the bit-slice 5 has a 1 bit for every page with a page signature where bit 5 is set). This drives both the updating of bitslices and their use in indexing.

You will need to modify the functions: newRelation() in reln.c, addToRelation() in reln.c, and

findPagesUsingBitSlices() in bsig.c. The modifications to newRelation() are relatively straightforward, but remember to update the relation parameters appropriately.

The addToRelation() should take a tuple, produce a page signature for it, then update all of the bit-slices corresponding to 1-bits in the page signature. This can be described roughly as follows:

<table width="656">

 <tbody>

  <tr>

   <td width="656">PID = data page where new Tuple insertedPsig = makePageSig(Tuple) for each i in  0..pm-1 {     if (Psig bit[i] is 1) {Slice = get i’th bit slice from bsigFileset the PID’th bit in Slicewrite updated Slice back to bsigFile     }}</td>

  </tr>

 </tbody>

</table>

The findPagesUsingBitSlices() function computes a page-level signature for query and then takes an intersection of the bitslices corresponding to the 1-bits in the page signature. This gives a “matching” pages list straight away, and hopefully after reading far less of the <em>Rel</em>.bsig file than would be read using a <em>Rel</em>.psig file. The method can be described roughly as follows:

<table width="656">

 <tbody>

  <tr>

   <td width="656">Qsig = makePageSig(Query)Pages = AllOneBits for each i in 0..pm-1 {     if (Qsig bit[i] is 1) {Slice = get i’th bit slice from bsigFile         zero bits in Pages which are zero in Slice     }}</td>

  </tr>

 </tbody>

</table>

<strong>Testing</strong>

The following simple tests provide a sanity-check for your code, once you’ve got it sufficiently implemented to execute (at least partially) the insert and select commands.

<table width="656">

 <tbody>

  <tr>

   <td width="656"># make a file to hold 10000 6-attribute tuples$ <strong>./create R 10000 6 1000</strong># make some data, and save it in a file$ <strong>./gendata 10000 6 1234567 13 &gt; R.in</strong># load tuples from R.in into files of relation R$ <strong>./insert R &lt; R.in</strong># check the structure of R’s files$ <strong>./stats R</strong> Global Info: Dynamic:#items:  tuples: 10000  tsigs: 10000  psigs: 137  bsigs: 6304#pages:  tuples: 137  tsigs: 27  psigs: 28  bsigs: 28Static:tups   #attrs: 6  size: 56 bytes  max/page: 73   sigs   bits/attr: 9tsigs  size: 88 bits (11 bytes)  max/page: 372   psigs  size: 6304 bits (788 bytes)  max/page: 5   bsigs  size: 144 bits (18 bytes)  max/page: 227# linear scan of all data (i.e. run an open query; ignore signatures)# if you want to see alll 10000 tuples, don’t pipe through tail $ <strong>./select R ‘?,?,?,?,?,?’ x | tail -6</strong># search for a tuple by the first attribute (not using signatures)$ <strong>./select R ‘1234999,?,?,?,?,?’ x</strong>1234999,UEkVEljYuGrloQCzLjmw,a3-183,a4-100,a5-017,a6-432Query Stats:# sig pages read:    0# signatures read:   0# data pages read:   137# tuples examined:   10000# false match pages: 136</td>

  </tr>

 </tbody>

</table>

# search for a tuple by the first attribute (using tuple signatures)

<h2>$ ./select R ‘1234999,?,?,?,?,?’ t</h2>

1234999,UEkVEljYuGrloQCzLjmw,a3-183,a4-100,a5-017,a6-432

Query Stats:

# sig pages read:    27

# signatures read:   10000

# data pages read:   5

# tuples examined:   365

# false match pages: 4

# search for a tuple by first attribute (using page signatures)

<h2>$ ./select R ‘1234999,?,?,?,?,?’ p</h2>

1234999,UEkVEljYuGrloQCzLjmw,a3-183,a4-100,a5-017,a6-432

Query Stats:

# sig pages read:    28

# signatures read:   137

# data pages read:   1

# tuples examined:   73

# false match pages: 0

# search for a tuple by first attribute (using bit-sliced signatures)

<h2>$ ./select R ‘1234999,?,?,?,?,?’ b</h2>

1234999,UEkVEljYuGrloQCzLjmw,a3-183,a4-100,a5-017,a6-432

Query Stats:

# sig pages read:    7

# signatures read:   9

# data pages read:   1

# tuples examined:   73

# false match pages: 0

# check for expeected answers

<h2>$ grep ‘a3-241,a4-158,a5-407’ R.in</h2>

1237049,ovnsbtUWihCcCEoRWKcF,a3-241,a4-158,a5-407,a6-490

1242029,eptevNjxFwayfSGeFKrO,a3-241,a4-158,a5-407,a6-490

# search for tuples via several attributes (using no signatures)

<h2>$ ./select R ‘?,?,a3-241,a4-158,a5-407,?’ x</h2>

1237049,ovnsbtUWihCcCEoRWKcF,a3-241,a4-158,a5-407,a6-490

1242029,eptevNjxFwayfSGeFKrO,a3-241,a4-158,a5-407,a6-490

Query Stats:

# sig pages read:    0

# signatures read:   0

# data pages read:   137

# tuples examined:   10000

# false match pages: 135

# search for tuples via several attributes (using tuple signatures)

<h2>$ ./select R ‘?,?,a3-241,a4-158,a5-407,?’ t</h2>

1237049,ovnsbtUWihCcCEoRWKcF,a3-241,a4-158,a5-407,a6-490

1242029,eptevNjxFwayfSGeFKrO,a3-241,a4-158,a5-407,a6-490

Query Stats:

# sig pages read:    27

# signatures read:   10000

# data pages read:   2

# tuples examined:   146

# false match pages: 0

# search for tuples via several attributes (using page signatures)

<h2>$ ./select R ‘?,?,a3-241,a4-158,a5-407,?’ p</h2>

1237049,ovnsbtUWihCcCEoRWKcF,a3-241,a4-158,a5-407,a6-490

1242029,eptevNjxFwayfSGeFKrO,a3-241,a4-158,a5-407,a6-490

Query Stats:

# sig pages read:    28

# signatures read:   137

# data pages read:   2

# tuples examined:   146

# false match pages: 0

# search for tuples via several attributes (using bit-sliced signatures)

<h2>$ ./select R ‘?,?,a3-241,a4-158,a5-407,?’ b</h2>

1237049,ovnsbtUWihCcCEoRWKcF,a3-241,a4-158,a5-407,a6-490

1242029,eptevNjxFwayfSGeFKrO,a3-241,a4-158,a5-407,a6-490

Query Stats:

# sig pages read:    17

# signatures read:   27

# data pages read:   2

# tuples examined:   146

# false match pages: 0

# etc etc etc etc etc (think of more tests)

Based on the above, you should be able to devise other tests to check whether your select is producing the correct answers, and whether it’s producing the same number of signature reads and signature page reads. If it reads extra data pages using the same false match probability, that’s not catastrophic; however, reading more data pages is sub-optimal and will be penalised. With different false match probabilities and the same data, you would expect different numbers of pages to be read.

If you want to calculate the overall costs of the above methods, you should consider the sum of the page reads (both signature and data pages). The best method is one that minimises this cost (e.g. read less signature pages, but read no more data pages). You can tune the search methods by changing the false match probability; a higher false match probability produces smaller signatures, but results in more false-match pages being read.

Make sure you test your code on the CSE servers before you submit. It might work on your laptop, but there might be portability issues in moving it to the CSE servers. We test your code on the CSE servers; if it doesn’t work there, it doesn’t work as far as we’re concerned.