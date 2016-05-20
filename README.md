# demo_coadd_decam


Quick demo on making a coadd from the HiTs:

First download the data. 
Warning (this is 142GB)

```
curl -O https://lsst-web.ncsa.illinois.edu/~yusra/calexp_dir.tar.gz
tar xzf calexp_dir.tar.gz
```


Install the stack either using `lsstsw`: http://developer.lsst.io/en/latest/build-ci/lsstsw.html  or `eups distrib install` https://confluence.lsstcorp.org/display/LSWUG/Building+the+LSST+Stack+from+Source

If you are using distrib install, you can find the latest release here:
https://sw.lsstcorp.org/eupspkg/tags/?C=M;O=D
Clone the camera package corresponding to DECam

```
git clone https://github.com/lsst/obs_decam.git
```

Set up the right packages
```
export TAG=w_2016_20
setup -r obs_decam -t $TAG
```


# Single Frame Measurement and Calibration
Before, you make a coadd, you need to create the calexps. These have already been created, but if you'd like to reprocess:

```
processCcd.py calexp_dir --output calexp_out_dir  --id visit=0288976 ccdnum=50 --config calibrate.doPhotoCal=False calibrate.doAstrometry=False --configfile noIsrConfig.py --clobber-config
```

The config file contains a redirect, to an `IsrTask` that does nothing. We are starting wil images that have already had the instrument signature removed.


# Co-addition

First, make a `SkyMap`

```
makeDiscreteSkyMap.py calexp_dir --id --output coadd_dir
```

This `SkyMap` is now stored in coadd_dir/deepCoadd and  the contained geometry will be used for all coaddition steps. Next, run `makeCoaddTempExp` which will find all the calexps in "calexp_dir" that overlap the patch id supplied on the command line and warp them to the geometry defined in the SkyMap. 

```
makeCoaddTempExp.py calexp_dir --output coadd_dir --selectId --id filter=g tract=0 patch=11,8

assembleCoadd.py coadd_dir --selectId --id tract=0 patch=10,8 filter=g --config doInterp=False doMatchBackgrounds=False --output coadd_dir --clobber-config
```


Now run detection and measurement:

```
detectCoaddSources.py  coadd_dir --id tract=0 patch=11,8 filter=g  --output coadd_dir
mergeCoaddDetections.py coadd_dir --id tract=0 patch=11,8 filter=g --output coadd_dir
measureCoaddSources.py coadd_dir/ --id tract=0 patch=11,8 filter=g --config measurement.doApplyApCorr=yes --output coadd_dir
mergeCoaddMeasurements.py coadd_dir/ --id tract=0 patch=11,8 filter=g --output coadd_dir
```

forcedPhotCcd.py coadd_dir/ --id tract=0 patch=10,8 filter=g --config measurement.doApplyApCorr=yes --output coadd_dir

