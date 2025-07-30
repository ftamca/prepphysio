# prepphysio

Convert Siemens MR physiological data logs for analysis

Requires: Perl

Usage: prepphysio [options] \<physio file prefix>

 Preprocess Siemens physiological data dump files and convert them into a
 simpler format usable by AFNI and pretty much any other software.

 \<physio file prefix> is the common prefix of the physio data files. Do not
 include the .ecg/.ext/.puls/.resp suffix. The output file names will have a
 .1D suffix and will contain only the physio data acquired during the run
 (omitting physio data before and after the run). The outputs may be used
 directly for RETROICOR in AFNI and are easily read by other software as well.

 NOTE: The data should come from a SINGLE RUN (scan). Multiple runs in one
 physio data dump are not separated by this program and are a bad idea anyways
 because of limitations of the physio data collection system (however, see the
 examples and the -f, -l, -d and -D options for some ideas to help you deal with
 such data). Wait a few seconds after the end of a scan before stopping the
 physio data recording, to allow the buffered data from the last few seconds of
 the scan to be written to the file. The recording must be stopped to complete
 the log file, otherwise the incomplete data will be missing necessary timing
 information for processing.

 NOTE 2: This version is compatible with VD13 and above. Obtain an older
 version for use with VB17.

 NOTE 3: With VD13 and VE11, recording is limited to ~5min if ECG recording is
 included. Stop the ECG logging if you are not using ECG (e.g. start all signals
 and then immediately stop ECG in ideacommandtool).

 The [options], which must appear before \<physio file prefix>, include:

 -d  Disable TR consistency check. Mostly for troubleshooting. Unless you are
     deliberately varying the TR, which is unlikely, failing this sanity check
     means the data come from multiple runs or there is something wrong with
     the trigger source or sampling equipment.

     If you disable this check, the duration of the output files may not match
     the duration of the run. It may be close enough if you are sure that the
     input data are from a single run, but you must check the length of the
     output before you use it.

 -D  Don't even check the external triggers; only temporally align the physio
     signals. Trim the data only based on start time, not based on external
     triggers. The external trigger log is still required, but don't die if
     there were actually no triggers due to hardware not connected or
     unsupported pulse sequence. Not recommended, but occasionally useful.

     You are on your own as to synchronizing with any imaging data. Consider
     using -x to see if there are any useful triggers, and -v for start and
     end time stamps to help you (look for similar timestamps in DICOM image
     files). Implies -d.

 -e  Output ECG data too. ECG data are ignored by default; we prefer pulse.

 -f \<n>  Skip the first \<n> triggers. Use this to discard physio data acquired
     during a pre-scan period when images were not acquired (despite triggering)
     or where those images will be discarded before any physio correction. The
     goal is to be left with only the physio data associated with the images
     that are to be processed. See also -l and Example 2.

 -l \<n>  Skip the last \<n> triggers. Use this to discard physio data acquired
     during a post-scan period when images were not acquired (despite
     triggering) or where those images will be discarded before any physio
     correction. The goal is to be left with only the physio data associated
     with the images that are to be processed. See also -f and Example 2.

 -F \<n>  Keep the first \<n> triggers only and skip the remainder. Use this to
     keep only the data at the beginning when the rest is irrelevant, for
     example if you forgot to stop the physio recording. The goal is to be left
     with only the physio data associated with the images that are to be
     processed. See also -f and -l which are processed first.

 -o  Overwrite output files that already exist. The new default is to quit
     rather than to overwrite files you might not want to overwrite.

 -q  Be quiet. Don't print informational messages.

 -r  Output raw pulse trace rather than triggers. Pulse triggers are output by
     default, while respiration and ECG are output as raw traces.

 -t  Truncate the output so that the maximum length is a multiple of the
     detected TR.

     Sometimes, due to delays or asynchrony in the MR and/or physio acquisition,
     the apparent TR/sampling rate varies slightly and there will be more (or
     less) samples than you anticipate. The variation is usually only a few
     samples over the duration of a typical run, but I have seen several when
     using a short TR with the SMS-InI sequence, to name one example. These
     samples are retained because they are indeed part of the acquistion period.

     Some tools (like 3dretroicor) accommodate this variation without trouble.
     Some tools (like AFNI's ricor) have trouble, particularly when there are
     enough "extra" samples that the tool thinks you are trying to process an
     "extra" TR, which does not exist in the MR data. To deal with the
     latter case, the -t option truncates the physio outputs to the expected
     multiple of TR. Currently, if there are fewer samples than a full TR, the
     outputs are not padded out, but this is being considered (send feedback).

 -v  Be verbose. Mostly for troubleshooting.

 -x  Output the external trigger too. Mostly for troubleshooting. Helps to
     troubleshoot triggering issues, e.g. due to accidentally recording more
     than one run.

 One further option that may be useful: --help (to display this message).

Example 1: prepphysio myfmri-run1

 Turns files named myfmri-run1.{ext|puls|resp} into files named
 myfmri-run1.{puls|resp}.1D, which can be used in AFNI, etc. This is the
 most common usage for FMRI processing.

Example 2: prepphysio -l 3 myfmri+postscan

 Let's say your fMRI run was immediately followed by a 3-timepoint reference
 scan which was not part of the fMRI run but nevertheless ended up being
 included in the physio data recording. Skipping the last 3 triggers with -l3
 isolates the physio data of the fMRI run proper.

 This is also a way to separate two whole fMRI runs that were accidentally
 scanned in one physio data file: Run prepphysio first with -l or -F to skip the
 triggers associated with the second run, and rename the outputs for use with
 the first fMRI run. Then run prepphysio again with -f to skip the triggers
 associated with the first run, and rename the outputs for use with the second
 fMRI run.
