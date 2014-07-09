****************
Mails
****************

L1Calo
======

1. General information
----------------------

Hi Sasha,

Sorry for not getting in contact earlier! It's been a somewhat busy time for me lately, which hasn't left me much time free for thinking.

Anyway, we should try to get you integrated with our offline effort. Ideally we should have a chat, but it's getting late today and I don't know whether I'll have any time tomorrow (I'll be spending most of tomorrow running around, as much literally as figuratively!). So will at least try to explain the situation in an email to get started, and we should try to have a talk on Monday if we can't manage before.

So, first things first, what is your CERN login ID? I'll need that to give you write access to the SVN repositories.

Secondly, I'm guessing you've already found what passes for documentation in Atlas. The best starting point is probably https://twiki.cern.ch/twiki/bin/view/AtlasComputing/WebHome - the "workbooks" should contain some useful stuff (I refer to the software development workbook every now and then just to make sure I've got the syntax of some lesser-used commands right). Since Athena is derived from Gaudi a lot of the concepts should be familiar, but having never used LHCb software I don't know how many differences there are.

I'm also hoping someone has given you a copy of the L1Calo TDR - bit old now, but useful to explain how the system worked in Run 1 (bar a few things which subsequently changed). The L1Calo TWiki page, https://twiki.cern.ch/twiki/bin/view/Atlas/LevelOneCaloTrigger, also contains some useful links.

As for L1Calo, most of our offline software can be found in the TrigT1 package:

https://svnweb.cern.ch/trac/atlasoff/browser/Trigger/TrigT1

Within this the packages we're most concerned with are:

* TrigT1CaloSim - the algorithms used in MC simulation of the trigger

* TrigT1CaloTools - tools (technically AlgTools, if you want to search the Atlas TWiki for documentation) which are used by the algorithms to carry out many of their tasks.

* TrigT1CaloToolInterfaces - virtual interfaces to the tools (so the package using the tools doesn't have to depend directly on the package containing the tools - helpful in avoiding "dependency loops" when building the software).

* TrigT1CaloEvent - event data objects, both those representing readout data and temporary ones used only in simulation

* TrigT1CaloUtils - utility classes which don't fit the above categories. These range between little utilities for performing compression as we used in the Run 1 MET trigger through to functions which actually carry out the EM/Tau or Jet algorithms for a single window.

* TrigT1CaloMonitoring and TrigT1CaloMonitoringTools - the names are fairly self-explanatory. These are for monitoring we carry out in the Tier-0 farm during data-taking, both performance monitoring (e.g. "is the ET scale correct?", "what dead towers are there?") and diagnostic monitoring (use the simulation tools to predict what the module outputs should be and compare - this is why the simulation uses a set of AlgTools rather than just being part of the simulation algorithms themselves, so that the monitoring algorithms can use the same tools).

* TrigT1EventAthenaPool and TrigT1EventTPCnv - note no "calo" in those name, so these packages include Muon and Central trigger data as well. These are concerned with storing objects persistently on disk and reading them back into Athena. Atlas uses a scheme where each stored object has a "persistent" form, written to disc, and a "transient" form used in the job. If we change the content of the active (transient) object the TP convertor needs to be updated to fill the new version from the stored data. If the changes also change the data we wish to store, we create a new persistent object, and the convertor has to be able to fill the current transient form from any of the persistent forms (as far as possible).

* TrigT1CaloByteStream - "bytestream" is the raw data format, i.e. the data recorded directly from the Read Out Buffers (as opposed to stored objects, which the previous 2 packages are concerned with). This package contains the "bytestream convertors", which read the bytestream and unpack it into Athena objects. Since we also want to be able to simulate the raw data, these convertors also include tools to create a simulated bytestream from the (transient) Athena objects representing the readout data.

* TrigT1Interfaces - contains utility classes used for accessing/using L1 data (Calo, Muon or CTP) generally. For example, objects which allow the High-Level Triggers (HLT) to extract information from our Regions of Interest (RoIs), which contain data describing an object found by L1 packed into a single 32 bit word. Or functions used by these objects or our own algorithms for unpacking those words, various parameters which may be needed in different places (such as maximum number of trigger thresholds), etc.

There are other packages here which may in time concern us (calibration packages, mapping tools), but that's probably more than enough to start with!

The simulation routines also have a number of connections to other parts of the system. The obvious ones are the calorimeters (which provide our input data), the Central Trigger Processor (CTP) which we provide data to, and the Trigger Menu/Configuration, which provides us with many of the trigger settings.

Now most of this stuff has been pretty stable for a few years (monitoring getting more updates than other stuff), but for Run 2 we are changing how things work. I don't know what info Steve/Juraj gave you, but in outline here is what the system has done so far (and hence what we've been simulating and monitoring) and what is changing:

Run 1
-----
PreProcessor (represented by algorithm TriggerTowerMaker) takes calorimeter signals, digitises, identifies which crossing signals come from, sends calibrated 8 bit ET value to both Cluster Processor (EM/Tau) and Jet/Energy Processor (Jets, Missing ET, Sum ET).

Processor modules (Cluster Processor Module, Jet/Energy Module) find local objects (EM/Tau/Jet) passing different thresholds, count how many pass each threshold, send multiplicities for each threshold via backplane to a Common Merger Module. JEMs also form Ex, Ey, ET sums which they send to a different CMM. Also these modules provide Regions of Interest to the HLT if the event is accepted - the RoI tells the HLT the coordinates, which type it was (EM/TAU or Jet) and which thresholds it passed (EM/TAU are a single RoI containing information on both types of threshold).

CMMs form global (system-wide) counts of objects passing each threshold and pass this information to the CTP. Also the energy CMM forms global Ex, Ey and ET sums and applies trigger thresholds, again sending results to the CTP. The energy CMM also provides an "EnergySum RoI" to the HLT, containing 3 words which summarise the MET and SumET triggers (and something called "missing ET significance", formed from a combination of MET and SumET).

Run 2:
------
Preprocessor will use different types of digital filtering, can apply a bunch-crossing dependent baseline correction (as pileup effects are not uniform through a bunch train), and can use different calibrations for towers sent to the CP and JE systems. Naturally this changes the readout formats as well as the processing.

Processor modules still find local objects, but rather than counting the number passing each threshold they will now just pass the coordinates, the ET, and for EM/Tau a few bits indicating how isolated then are, directly to the Merger (now called a CMX rather than a CMM). The isolation algorithms will be somewhat different from Run 1 (so needs menu changes). They will still provide RoI data, but these will now contain the ET and isolation results rather than the list of thresholds passed, and EM and Tau RoIs will now be separate objects. The Ex/Ey/ET summing in the JEM is unchanged, except that we don't need to compress data before sending to the CMX.

CMXs will apply trigger thresholds, count multiplicities of objects passing each, and send that information to the CTP (more thresholds possible than in Run 1). They also send the individual object data (now called "Trigger Objects" or "TOBs") to the new topological processor. The energy CMX will still form MET/SumET/MEtSig triggers as before, and these values are sent to the Topo processor as well as the results being sent to the CTP.

So, some of the algorithms change, quite a bit of the readout data changes, and the data sent to the CTP and HLT are different. We need to be able to simulate this, read the data back into Athena, and perform monitoring. We also need to work with the HLT/CTP/Menu groups to make sure that all of the systems are expecting the same things and can handle the data provided, and of course our own online people to make sure that the headers of simulated datastreams match what the hardware will produce.

So, that's an outline of what we are doing at the moment. I should provide a summary of where we are currently & what needs doing, so we can work out where you can help best, but as this mail is long enough I think I should probably put that in a different one (plus my daughter is going to arrive any minute and expect to be fed!).

2. Bytestream decoder
---------------------

Learning the bytestream decoder stuff would be great, as that was something your predecessor (Pete Faulkner) always handled. It's probably also some of the most technical code we have, since it's concerned with unpacking the raw data (which is, as the name implies, a stream of bytes) into Athena objects and vice-versa, and the way some of that stuff is packed is more concerned with efficiency in the readout driver (ROD) than human intelligibility. Steve knows those formats better than I do.

Pete did leave some updated bytestream decoder code to handle some of the newer data formats, but some of them were not defined at the time when he left, most notably the trigger tower readout from the new MCM. That latter will also require a redefined Athena object to be unpacked into. I do have some thoughts on that, so we should probably come back to that one. Just learning how the existing stuff works will probably take a while. I will however copy Pete's newer stuff to CERN for ease of access (I've not committed it to the repository because until I had written more of the simulation I didn't have an easy way of testing much of it, though I think I could do so now - probably not today though as I'm supposed to be doing other stuff).

3. Pete's code
--------------

Hi Sasha,

A quick thing, as I'm actually supposed to be elsewhere at the moment.

I've copied Pete's code to ~watsona/public/athena/PJWF. It's the entire working directory he left, & is based on 17.0.3.6 and nothing tested in release 19. But it includes versions of TrigT1CaloEvent and TrigT1CaloByteStream where he has added classes for new data objects as well as the originals (and by the look of it kept copies of original versions of some of the code), and I think there is also some new stuff in TrigT1CaloMonitoring. I've actually not checked whether there are modifications to TrigT1CaloSim to let him test the new stuff.

I should pick out the new classes and add them to the repository (and some of my new simulation code), but have to disappear offline now. But if in addition to learning the existing stuff you want to look at some of the additions (anything with "CMX" or "Tob" in the name is new) then at least it's there.

4. Data formats
---------------

Hi Sasha,

As I said on Skype, it'll be useful to have a description of the bytestream formats that we are decoding. For the Run 1 data (i.e. the code that's currently in svn) the best I have is the ReadOut Driver (ROD) documentation. You can find this in the "module descriptions" here: https://atlas-l1calo.web.cern.ch/atlas-l1calo/html/orgweb/Modules/Modules.html#ROD - I'm hoping you can access that even if they've still not sorted the TWiki, otherwise let me know and I'll send you copies of the documents.

For the Run 2 formats the best I have are the 2 attached spreadsheets (Steve, is there other documentation?). Here "G-Link" are the inputs to the ROD from the different modules in the system, "S-Link" are the outputs from the ROD to the ReadOut System (ROS). The bytestream is based on the S-Link data (plus headers etc, the principle of which is described in the ROD specification).

As far as the Run 1 bytestream goes, the various module readouts are unpacked (and, in simulation, packed) into the following Athena objects (from the package Trigger/TrigT1/TrigT1CaloEvent):

* PreProcessor readout (trigger towers):
  - TriggerTower = pair of readout channels, EM+Had at same eta, phi
  - TriggerTowerCollection = container class for TriggerTowers in StoreGate

When we talk about a "trigger tower" in hardware or calibration we mean EM _or_ Hadronic. The object TriggerTower contains 2 "trigger towers". This is something I actually would like to change in Run 2 (it's historical, and while convenient for one task is inconvenient for several others).

* CPM readout:
  - CPMHits = EM/Tau hit sums (multiplicity for each threshold) within each module (obsolete in Run 2)
  - CPMRoI = L1Calo DAQ copy of EM/Tau Region of Interest
  - CPMTower = towers received by CPMs from PreProcessor
  - CPMHitsCollection, CPMRoICollection, CPMTowerCollection = container classes

* JEM readout:
  - JEMHits = jet hit sums within each module (obsolete in Run 2)
  - JEMRoI = L1Calo DAQ copy of Jet RoI
  - JetElement = jet elements received by JEMs from PreProcessor  (jet element = trigger towers summed to coarser granularity in PPr)
  - JEMEtSums = Ex, Ey, ET sums for a JEM
  - JEMHitsCollection, JEMRoICollection, JetElementCollection, JEMEtSumsCollection = container classes

* CMM readout (obsolete in Run 2, replaced by CMX readout):
  - CMMCPHits = EM/Tau hit multiplicity sums for CMMs in CP system (crate or global sums)
  - CMMJetHits = Jet hit multiplicity sums for Jet CMMs (crate or global sums)
  - CMMEtSums = Ex, Ey, ET sums from Energy CMMs (crate or global sums)
  - CMMRoI = Missing ET and Sum ET RoI (3 words). Plus JetEtSum RoI (1 word, obsolete in Run 2)
  - CMMCPHitsCollection, CMMJetHitsCollection, CMMEtSumsCollection

There are also classes "CPBSCollection", "JEPBSCollection" and "JEPRoIBSCollection", which collect together the different types of data contained in the bytestreams from the CP and JE systems, which is for the convenience of the bytestream decoders.

Many of these objects contain vectors of data, rather than single variables, to allow the possibility of reading out multiple time-slices of data (i.e. from one or more crossings before or after the triggered bunch crossing). In normal running we don't do this for most of these, but as the option exists in the readout the objects and decoders have to accommodate this.

There are also a couple of other classes in TrigT1CaloEvent which are used only within the simulation (e.g. EmTauROI, JetROI), and are not part of the readout data. I'm planning to phase these out for Run 2 (they're a hangover from the original design of the simulation). "JetInput" is an oddity, and also not part of the readout: it represents the inputs to the jet algorithm, i.e. a data transfer within the JEM, and is almost identical to the JetElement. The difference is in the Forward Calorimeter (FCAL), where we split each JetElement into 2 so that the jet algorithm can treat the phi granularity as being uniform between barrel/endcap and FCAL. For bytestream conversion purposes all of these classes can be ignored.

cheers,
     Alan

P.S. I mentioned that the PreProcessor readout/TriggerTower would change for Run 2. I don't have a description of the actual formats (I've found some slides from February, but doubt they are definitive), but the big changes are that we'll be subtracting a bunch-crossing-dependent baseline (and so need a record of the value used) and we are planning on reading 2 calibrated ET values per tower (separately for EM/Tau and Jet/ET triggers). Packing more data in means the actual packed (bytestream) format is likely to be significantly different from Run 1.

As mentioned above, one thing I'd like to do while we are rewriting this bit is to change the TriggerTower object so that 1 TriggerTower = 1 trigger tower (i.e. one PreProcessor readout channel) rather than two. It's partly aesthetics: since the TriggerTower contains both an EM and a Had tower it's a big, ugly object with 2 copies of everything. It's also partly practical, since while it's convenient for algorithm simulation (the reason it was originally written this way) it's inconvenient for readout decoding (the EM and Had towers come from different modules) and of no value for calibration studies or monitoring. And partly because it's simpler: one TriggerTower (object) representing 2 trigger towers (hardware/signals) is a good way of confusing people! It's not the most urgent thing, but if you are looking at the bytestream decoders I thought it best to mention that I've been thinking about making this change, since it should allow us to simplify that decoder in future.

This wouldn't affect CPMTower or JetElement, both of which also contain 2 layers. I think these are less of a problem as they are simpler objects, and for these both layers are read out from the same hardware module so the grouping is more natural. And as the trigger algorithm simulations, which use both layers, are actually based on these objects it's more useful for these to contain both layers in one object than it is for TriggerTower.

* https://drive.google.com/file/d/0Bw1zWDlANxEoRmVEc2tGLXU0bDdCcXdjMkZkTTVJUVJfQ0JR/edit?usp=sharing
* https://drive.google.com/file/d/0Bw1zWDlANxEoM1BaOURFSEV1YkdlQ3dKNTJyN3ZETkM4VENF/edit?usp=sharing


5. On formats from steve
-------------------------------

Hello,

those are indeed the current best referencess.  The
old ROD document, though of course outdated by the current
upgrade, has many useful descriptions and general principles
that still hold true in the new data formats, so it's still
a good place to start, remembering that some of it will
have changed, though not so much in the JEM and CPM area.

The PPM format is still a bit up in the air I believe.
I haven't seen a definitive proposal yet, but I think this
is going to be discussed soon.  The situation for the
Topo readout is similar!
      
Cheers, Steve