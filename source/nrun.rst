*******************
Dump run number
*******************

.. code-block:: sh

    $> asetup 19.1.2
    $> export STAGE_SVCCLASS=atlcal # user should hve access to the calibration runs (I use l1ccalib)

.. code-block:: python

    # Bytestream event format module
    import eformat as ef
    
  
    # Not working (#run == 0)
    bs = ef.istream("/castor/cern.ch/grid/atlas/DAQ/l1calo/00237155/data14_calib.00237155.calibration_L1CaloEnergyScan.daq.RAW._lb0000._SFO-1._0001.data")
    for obj in bs: print("#run=%d" % obj.run_no())
    
    # Working (#run != 0)
    bs = ef.istream("/castor/cern.ch/grid/atlas/DAQ/l1calo/00237991/data14_calib.00237991.calibration_L1CaloPprPedestalRunPars.daq.RAW._lb0000._ROSEventBuilder._0001.data")
    for obj in bs: print("#run=%d" % obj.run_no())
