### About JC Parser

The Johnson-Charniak parser is a tool for parsing natural language (in text form). 
It is used to construct parse trees.

[Mark Johnson's Software](http://http://web.science.mq.edu.au/~mjohnson/Software.htm) 

### Assumptions:
You have a fresh CentOS machine. You have a user account on this machine with sudo privileges. You are logged in as this user and are in the home directory.

### Setup
Install some necessary tools
 
    sudo yum install flex
    sudo yum install gcc-c++

Obtain the JC Parser source code

    wget http://web.science.mq.edu.au/~mjohnson/code/reranking-parser-2011-12-17.tgz
    tar -zxvf reranking-parser-2011-12-17.tgz

Note: There is a line missing in "reranking-parser/second-stage/programs/features/best-parses.cc". Add the following to best-parses.cc near other #includes.

    #include <unistd.h>

Compile

    cd /reranking-parser
    make
    
Note: The compiler may complain about an unsupported O level, in which case open the Makefile and edit the CFLAGS on line 65, the '-O6' flag to just '-O'.    
    

### Run the parser on a .txt file
  
    ./parse.sh sample-data.txt

The output should look like the following

    
    (S1 (S (NP (DT This)) (VP (AUX is) (NP (DT some) (NN text) (SBAR (IN that) (S (NP (PRP we)) (VP (MD should) (, ,) (ADVP (RB presumably)) (, ,) (ADJP (JJ parse))))))) (. !)))
    (S1 (S (NP (NP (DT A) (JJ second) (NN sentence)) (PRN (-LRB- -LRB-) (PP (ADVP (RB much)) (IN like) (NP (DT the) (JJ first))) (-RRB- -RRB-))) (VP (MD will) (ADVP (RB also)) (VP (VB help))) (. .)))
    (S1 (NP (S (NP (DT This) (NN sentence)) (VP (VBZ contains) (NP (NP (DT some) (ADJP (RB very) (JJ funny)) (NNS tokens)) (, ,) (PP (JJ such) (IN as) (UCP (ADJP (JJ \/)) (CC and) (ADJP (# #) (CC and) (NN %) (NN %)) (CC and) (ADJP (JJ *) (CC &) (JJ *)) (CC and) (. !) (. !) (FRAG (CC and) (NP (NN ~) (CC and) (NN +=) (NNS <)) (. .)) (. .))) (. ?)))) (: ;) (NP (NNP _)) (: -) (CC &) (NP (NNP >))))
  
