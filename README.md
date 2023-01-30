![GitHub Repo stars](https://img.shields.io/github/stars/roblabs/teqc?label=Source&style=social) ° ![GitHub](https://img.shields.io/github/license/roblabs/teqc)

> "TEQC (**Translation, Editing and Quality Checking**) — is [UNAVCO]-designed and supported for a variety of [GPS]/[GNSS] pre-processing and quick post-processing tasks, including translation to [RINEX] or BINEX, time windowing, satellite filtering, metadata correction, dataset merging or splitting, and GPS/GNSS quality checking of the data including coarse point-positioning."

> "[Teqc] (pronouced "tek") is a simple yet powerful and unified approach to solving many pre-processing problems with [GPS], GLONASS, Galileo, SBAS, Beidou, QZSS, and IRNSS data, especially in [RINEX] or BINEX format:"

---

## Definitions

The Teqc documenation has an excellent glossary of terms they use in the `teqc` software.  Of note for this blog post are:

* decimate - "*modulo decimation of OBS epochs to # time units*"
* [epoch] - "*a specific time instance, using the GPS time basis or the time basis from another constellation*"
* [GPS] - "*'**G**lobal **P**ositioning **S**ystem'; a specific spaceborne radionavigation system financed and operated by the U.S. Department of Defense*"
* [RINEX] - "*'**R**eceiver **In**dependent **Ex**change'; ASCII exchange representation of [GNSS] data and metadata*"
* [SV] - "*'**S**pace **V**ehicle', referring originally to a specific Navstar GPS satellite, but now used to refer to any one of the Navstar GPS, GLONASS, Beidou/Compass, Galileo, QZSS, IRNSS, or SBAS satellites*"

## Getting Started

Download binaries for macOS, Linux, Windows and others from the [Teqc](https://www.unavco.org/software/data-processing/teqc/teqc.html) site.  You can also find documentation & tutorials at that site.

```bash
teqc -version

# executable:  teqc
# version:     teqc  2019Feb25
```

---

## Example

### Example of converting [Leica](https://leica-geosystems.com/en-us) file in the `.m00` format.

#### `teqc` Help 

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

#### Convert Leica to RINEX

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

This reduces a *1 second* file from 32 MB to a *30 second* file down to 1.1 MB.  You can inspect the `.obs` file for `COMMENT` and see that the decimation is now 30 seconds.

*Analyze the RINEX output, focusing on decimation and excluded satellites.*

```console
ll -h *.obs

# result of Linux listing of OBS files to determine file size, human readable
-rw-r--r--  1 roblabs  staff   1.1M Jan  4 16:24 GNSS202235660778.30.obs
-rw-r--r--  1 roblabs  staff    32M Jan  4 16:16 GNSS202235660778.obs

# ---

cat GGNSS202235660778.30.obs | grep COMMENT

# result of Linux regular expression (grep)
teqc  2019Feb25                         20230105 00:16:58UTCCOMMENT
BIT 2 OF LLI FLAGS DATA COLLECTED UNDER A/S CONDITION       COMMENT
GNSS Survey                                                 COMMENT
GNSS Survey                                                 COMMENT
Project creator:                                            COMMENT
Leica Geosystems AG                                         COMMENT
 SNR is mapped to RINEX snr flag value [0-9]                COMMENT
  L1 & L2: min(max(int(snr_dBHz/6), 0), 9)                  COMMENT
Forced Modulo Decimation to 30 seconds                      COMMENT
```

---

### Decimate & GPS Only

Our goal here is to do two operations in the same `teqc` command:  _decimate_ & convert only the GPS data in the original file.

You can call `teqc` with multiple flags (or sometimes called switches).  This will be important if, for example, you want to convert with the following options:

1.  Convert to [RINEX]
2.  decimate down to 30 seconds
3.  Only use [SV]s from [GPS].  That is, _filter out_ other constellations such as GLONASS, etc.

First, we need to understand how to _filter out_ or remove from the RINEX the constellations that we don't want.

Let's look at how the help information from the `teqc` executable is displayed.

```
teqc -help | grep use\ any
```

        -G                   don't use any GPS SVs
        -R                   don't use any GLONASS SVs
        -S                   don't use any SBAS SVs
        -E                   don't use any Galileo SVs
        -C                   don't use any Beidou SVs
        -J                   don't use any QZSS SVs
        -I                   don't use any IRNSS SVs


The [Teqc Tutorial](https://www.unavco.org/software/data-processing/teqc/doc/UNAVCO_Teqc_Tutorial.pdf) from unavco.org has this documentation about excluding GLONASS(**R**), GALILEO(**E**), and SBAS(**S**), but leaving all GPS(G): 

```bash
# These flags can explicity used to remove some constellations.
# See page 12 of the PDF documentation for details
# -R                   don't use any GLONASS SVs
# -S                   don't use any SBAS SVs
# -E                   don't use any Galileo SVs
teqc -R -E -S +obs + +nav +,+ -tbin 1d tbinoutput inputfiles
```

#### Command

Now we can put together decmitate and save GPS only.

```bash
teqc -O.dec 30 -R -E -S -C -J GNSS202235660778.obs > GNSS202235660778.30.GPS.obs
```

#### Results

* As we saw with just [_decimate_](#decimate), we reduced a 32 MB *1-second* file down to a *30-second* file at 1.1 MB.
* Now with _decimate_ and _GPS Only_, we reduced a 32 MB file down to 423K.
* You can review the comments in the RINEX file that shows which satellites were excluded.

*Analyze the RINEX output, focusing on decimation and excluded satellites.*

```console
ll -h *.obs

# result of Linux listing of OBS files to determine file size, human readable
-rw-r--r--  1 roblabs  staff    32M Jan  4 16:16 GNSS202235660778.obs
-rw-r--r--  1 roblabs  staff   1.1M Jan  4 16:24 GNSS202235660778.30.obs
-rw-r--r--  1 roblabs  staff   423K Jan 12 18:11 GNSS202235660778.30.GPS.obs

# ---

cat GNSS202235660778.30.GPS.obs | grep COMMENT

# result of Linux regular expression (grep)
teqc  2019Feb25                         20230105 00:16:58UTCCOMMENT
BIT 2 OF LLI FLAGS DATA COLLECTED UNDER A/S CONDITION       COMMENT
GNSS Survey                                                 COMMENT
GNSS Survey                                                 COMMENT
Project creator:                                            COMMENT
Leica Geosystems AG                                         COMMENT
 SNR is mapped to RINEX snr flag value [0-9]                COMMENT
  L1 & L2: min(max(int(snr_dBHz/6), 0), 9)                  COMMENT
Forced Modulo Decimation to 30 seconds                      COMMENT
teqc edited: all GLONASS satellites excluded                COMMENT
teqc edited: all SBAS satellites excluded                   COMMENT
teqc edited: all Galileo satellites excluded                COMMENT
teqc edited: all Beidou satellites excluded                 COMMENT
teqc edited: all QZSS satellites excluded                   COMMENT
```

---

### `meta`

You can use the metadata extraction options `+meta`.

Interrogate the file native m00 file

```bash
teqc +meta GNSS202235660778.m00
```

Result:

> week: 2255

---

Once you have decimate'd the file & filtered the SVs for *GPS Only*, you can use `+meta` to get a summary of the RINEX file.


```console
teqc +meta GNSS202235660778.30.GPS.obs

# result of metadata extraction
filename:                GNSS202235660778.30.GPS.obs
file format:             RINEX
file size (bytes):       432666
start date & time:       2022-12-22 16:53:30.000
final date & time:       2022-12-22 21:17:30.000
sample interval:         30.0000
possible missing epochs: 0
4-char station code:     GNSS
station name:            -Unknown-
station ID number:       
antenna ID number:       -Unknown-
antenna type:            -Unknown-       NONE
antenna latitude (deg):   90
antenna longitude (deg):   0
antenna elevation (m):   -6356752.3142
antenna height (m):      2.0000
receiver ID number:      287551
receiver type:           LEICA
receiver firmware:       7.500/4.000
RINEX version:           2.11
RINEX translator:        teqc  2019Feb25
trans date & time:       2023-01-13 02:11:36.000
```

[epoch]:  https://www.unavco.org/help/glossary/glossary.html#epoch
[GNSS]:   https://www.unavco.org/help/glossary/glossary.html#GNSS
[GPS]:    https://www.unavco.org/help/glossary/glossary.html#GPS
[RINEX]:  https://www.unavco.org/help/glossary/glossary.html#RINEX
[SV]:     https://www.unavco.org/help/glossary/glossary.html#SV
[Teqc]:   https://www.unavco.org/software/data-processing/teqc/teqc.html
[UNAVCO]: https://www.unavco.org
