### Galaxy Developer Setup

This is a guide to set up a basic Galaxy install with none of the existing Alveo tools included. It is designed for third party developers to get up a clean installation in order to develop tools for possible inclusion in the Alveo Galaxy system. 

**Instructions (Ubuntu)**

Install required packages

    sudo apt-get install -y -q python2.7 python2.7-dev mercurial autoconf automake autotools-dev build-essential cmake git-core libatlas-base-dev libblas-dev liblapack-dev libc6-dev subversion pkg-config sudo wget

Clone Galaxy repoository

    hg clone https://bitbucket.org/galaxy/galaxy-dist/

Remove default Galaxy tools

    cd galaxy-dist
    vi config/tool_conf.xml

Paste in the following

```
<?xml version='1.0' encoding='utf-8'?>
<toolbox>
  <section id="getext" name="Get Data">
    <tool file="data_source/upload.xml" />
  </section>
  <section id="dev" name="Development">
    <!-- Add new tool references here -->

  </section>
</toolbox>
```

Start it up

    ./run.sh
