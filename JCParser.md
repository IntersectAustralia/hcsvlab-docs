### Background

The Johnson-Charniak parser is a tool for parsing natural language (in text form). It is used to construct parse trees.

[Mark Johnson's Software](http://web.science.mq.edu.au/~mjohnson/Software.htm) 

### Assumptions:
* You have a fresh CentOS machine. 
* You have a non-root user account on this machine with sudo privileges. 
* You are logged in as this user and are in the home directory.

### Setup
Install some necessary tools
 
    sudo yum install flex gcc-c++

Obtain the JC Parser source code

    cd ~
    wget http://web.science.mq.edu.au/~mjohnson/code/reranking-parser-2011-12-17.tgz
    tar -zxvf reranking-parser-2011-12-17.tgz

There is a line missing in `reranking-parser/second-stage/programs/features/best-parses.cc` - edit the file and add the following near other #includes:

    #include <unistd.h>

Compile

    cd ~/reranking-parser
    make
    
<b>Note:</b> The compiler may complain about an unsupported O level, in which case open the Makefile and edit the `CFLAGS` on line 65, the `-O6` flag to just `-O`.    
