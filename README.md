# demo_coadd_decam


Quick demo on making a coadd from the HiTs:

First download the data. 
Warning (this is 142GB)

```
curl -O https://lsst-web.ncsa.illinois.edu/~yusra/calexp_dir.tar.gz
tar xzf calexp_dir.tar.gz
```

Install the stack either using `lsstsw`: http://developer.lsst.io/en/latest/build-ci/lsstsw.html  or `eups distrib install` https://confluence.lsstcorp.org/display/LSWUG/Building+the+LSST+Stack+from+Source

Clone the camera package corresponding to DECam

```
git clone https://github.com/lsst/obs_decam.git
```

Set up the right packages
```
export TAG=w_2016_20
setup -r obs_decam -t $TAG
```

Before, you make a coadd, you need to create the calexps. These have already been created, but if you'd like to reprocess:

```
processCcd.py calexp_dir --output calexp_out_dir  --id visit=0288976 ccdnum=50 --config calibrate.doPhotoCal=False calibrate.doAstrometry=False --configfile decamconfig.py --clobber-config
```



Now, make a `SkyMap`

```
makeDiscreteSkyMap.py calexp_dir --id --output coadd_dir

makeCoaddTempExp.py calexp_dir --output coadd_dir --selectId --id filter=g tract=0 patch=11,8

assembleCoadd.py coadd_dir --selectId --id tract=0 patch=10,8 filter=g --config doInterp=False doMatchBackgrounds=False --output coadd_dir --clobber-config
```


