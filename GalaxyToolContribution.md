### Contributing Galaxy Tools

The [Alveo Galaxy Toolshed](http://galaxy.alveo.edu.au:9009) allows users to contribute tools for inclusion in Alveo's Galaxy instance. By uploading a tool to the Toolshed, it can then be looked at and tested by the Alveo team before being inluded in Galaxy for easy use with data from Alveo.

### Creating Tools

Tools should be created using documentation provided [here](https://wiki.galaxyproject.org/Admin/Tools/AddToolTutorial). In simplest form, a tool requires a tool definition file which describes the config and options of the tool and the tool script itself. 

An example tool definition file looks like this:
```
<tool id="fa_gc_content_1" name="Compute GC content">
  <description>for each sequence in a file</description>
  <command interpreter="perl">toolExample.pl $input $output</command>
  <inputs>
    <param format="fasta" name="input" type="data" label="Source file"/>
  </inputs>
  <outputs>
    <data format="tabular" name="output" />
  </outputs>

  <help>
This tool computes GC content from a FASTA file.
  </help>

</tool>
```
A full list of tool definition syntax can be seen [here](https://wiki.galaxyproject.org/Admin/Tools/ToolConfigSyntax)


### Uploading Tools to the Toolshed

The following steps detail how to upload a tool to the Toolshed:

1. Log into the [Toolshed](http://galaxy.alveo.edu.au:9009), creating an account if you don't already have an existing one
2. Click "Create New Repository" from the left hand side menu.
3. Enter a name and description for your repository. A repository is what Toolshed uses to store files and metadata for your tool. In the backend, it is actually a Mercurial repository, so additions and updates to your tool can be seen through changesets. 
4. Click "Upload Files to Repository" button from the top right of the repository page.
5. Upload files to repository. This can be done by uploading a tarball containing all files for your tool, or single files at a time. 
