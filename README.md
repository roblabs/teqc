# teqc

> *"Teqc (pronouced "tek") is a simple yet powerful and unified approach to solving many pre-processing problems with GPS, GLONASS, Galileo, SBAS, Beidou, QZSS, and IRNSS data, especially in RINEX or BINEX format:"*

Download binaries for macOS, Linux, Windows and others from [Unavco.org](https://www.unavco.org/software/data-processing/teqc/teqc.html).  You can also find documentation at that site.

```bash
teqc -version

# executable:  teqc
# version:     teqc  2019Feb25
```

---

## Example of converting Leica file in the `.m00` format.

### `teqc` Help 

You can filter help results for *leica*.

```bash
teqc -help | grep lei
```

```console
 	-lei[ca] code        input is from Leica receiver, record type 'code':
	                       code = ds for Leica DS format
	                       code = lb2 for Leica LB2 format
	                       code = mdb for Leica MDB format
```

---

### `meta`

Interrogate the file native m00 file

```bash
teqc +meta GNSS202235660778.m00
```

Result:

> week: 2255

---

### Convert Leica to RINEX

```bash
# The output RINEX file could be `.obs`
# You may also see it in the format of `.yyo`, where yy = 22.  E.g, `.22o`

# Use either of these commands.  The record type is `mdb`
teqc -lei   mdb GNSS202235660778.m00 > GNSS202235660778.obs
teqc -leica mdb GNSS202235660778.m00 > GNSS202235660778.obs
```

```bash
     2.11           OBSERVATION DATA    M (MIXED)           RINEX VERSION / TYPE
teqc  2019Feb25                         20230105 00:07:30UTCPGM / RUN BY / DATE
OSX ker:10.11.6|Core i5|gcc 4.3 -m64|OSX ker:10.10+|=+      COMMENT
BIT 2 OF LLI FLAGS DATA COLLECTED UNDER A/S CONDITION       COMMENT
-Unknown-                                                   MARKER NAME
-Unknown-           -Unknown-                               OBSERVER / AGENCY
287551              LEICA               7.500/4.000         REC # / TYPE / VERS
-Unknown-           -Unknown-       NONE                    ANT # / TYPE
        0.0000        0.0000        0.0000                  APPROX POSITION XYZ
        2.0000        0.0000        0.0000                  ANTENNA: DELTA H/E/N
     1     1                                                WAVELENGTH FACT L1/2
     6    L1    L2    C1    P2    S1    S2                  # / TYPES OF OBSERV
     1.0000                                                 INTERVAL
GNSS Survey                                                 COMMENT
GNSS Survey                                                 COMMENT
Project creator:                                            COMMENT
Leica Geosystems AG                                         COMMENT
 SNR is mapped to RINEX snr flag value [0-9]                COMMENT
  L1 & L2: min(max(int(snr_dBHz/6), 0), 9)                  COMMENT
  2022    12    22    16    53   18.0000000     GPS         TIME OF FIRST OBS
    18                                                      LEAP SECONDS
                                                            END OF HEADER
 22 12 22 16 53 18.0000000  0 22G31G22G03G04G21G01G26G32G16S31S33S35
```

---

### decimate

Decimate the file from 1 sec to 30 seconds.

```bash
teqc -help | grep dec
```

>		`-O.dec[imate] interval[:offset]          modulo decimation of OBS epochs to interval time units (default in seconds),`

```bash
teqc -O.dec 30 GNSS202235660778.obs > GNSS202235660778.30.obs
```

#### Results

This reduces a *1 second* file from 32 MB to a *30 second* file down to 1.1 MB.

> *Forced Modulo Decimation to 30 seconds                      COMMENT*

```console
-rw-r--r--  1 roblabs  staff   1.1M Jan  4 16:24 GNSS202235660778.30seconds.obs
-rw-r--r--  1 roblabs  staff    32M Jan  4 16:16 GNSS202235660778.obs
```
