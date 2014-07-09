Shell commands
=====================


* `Setup the ATLAS environment <https://twiki.cern.ch/twiki/bin/view/AtlasComputing/AtlasSetup?redirectedfrom=Atlas.AtlasSetup>`_ if your account n **NOT** in *zp* group (e.g, on the Birmingham cluster)

.. code-block:: sh


    $> export ATLAS_LOCAL_ROOT_BASE=/cvmfs/atlas.cern.ch/repo/ATLASLocalRootBase
    $> source /cvmfs/atlas.cern.ch/repo/ATLASLocalRootBase/user/atlasLocalSetup.sh

* If your account **IN** zp group

.. code-block:: sh
    
        $> setupATLAS

* Setup athena environment
  
.. code-block:: sh

    $> asetup 17.2.8.8,32,slc5,here
    or 
    $> asetup 18.1.2.1,slc6,here

**here** is needed if you would like to setup development environment in the current directory


* Get a package in the athena development environment

.. code-block:: sh

    $> cmt show packages | grep -i YourPackageName
    ... # extract the revision number of the package (tag name)
    $> cmt co -r YourRevisonNumber $SVNROOT/YourProject/YourPackagePath
 
* Compile project

.. code-block:: sh

    $> cd YourProject/YourPackagePath/cmt
    $> cmt config
    $> source setup.sh
    $> cmt make -j
