# Platform

Quorum has been tested on Linux with gcc 4.4 to gcc 4.7.

# Installation


You should download the latest release distribution tar ball from the
[releases section](https://github.com/gmarcais/Quorum/releases). If
compiling from the github code, you will need
[yaggo](https://github.com/gmarcais/yaggo/releases) as well.

Quorum requires [Jellyfish](https://github.com/gmarcais/Jellyfish/releases) to be installed.
For Quorum to compile and
run properly, `pkg-config` must find Jellyfish and the library loader
must find the shared library. See the [README of Jellyfish]](https://github.com/gmarcais/Jellyfish/blob/master/README.md)
for details.

Provided that Jellyfish is installed and accessible, install the usual
way:

~~~
./configure --prefix=/path/where/to/install
make
make install
~~~

Note that 'make install' is necessary or the paths coded in the quorum
scripts will not be valid.

# Usage

Only one switch (`-s`) is required to run Quorum. This switch specify
the size of the Jellyfish hash and it must be large enough so that all
k-mers will fit into memory. With Illumina reads, a good estimate for
this size is:

  (G + k * n) / 0.8

where G is the estimated genome size, k is the k-mer length (24 by
default) and n is the number of reads. If the chosen size is too
small, quorum will stop with the error message: "Failed: Increase the
size parameter".

For example, for a bacteria with 2 million Illumina reads in files
read1.fastq and read2.fastq, the command would be:

~~~
quorum -s 50M read1.fastq read2.fastq
~~~

The output corrected file is called by default `quorum_corrected.fa`.

#Output format

The correction made are appended to the header line in the fasta
format. For example, the following 101 bases long read:

~~~
  @1204
  GACCGGGCATGGGCTGAGCCTGTTCGGGAAGCTGACGGAGCCGGAAGAGGCCGGGATCGACCCTTCCGCCCCGCCCGCCGACTGGGTCGACCGGCCGGGCG
~~~  

is corrected to:

~~~
  >1204 86:sub:T-C 91:3_trunc 62:5_trunc
  CTTCCGCCCCGCCCGCCGACTGGGCCGAC
~~~

The coordinate system is 0-based in the original reads (like a C or
Perl array). Here, at base 86 a substitution was made from T to C. The
5\_trunc is the index of the first base (0 if not specified) and the
3\_trunc is the index after the last base (read length if not
specified). Hence, the length of the corrected reads is computed as
3\_trunc - 5\_trunc (29 in this example). The uncorrected and corrected
reads align as follows:

~~~
0                                                            62                      86   91        101
|                                                             |                       |    |         |
GACCGGGCATGGGCTGAGCCTGTTCGGGAAGCTGACGGAGCCGGAAGAGGCCGGGATCGACCCTTCCGCCCCGCCCGCCGACTGGGTCGACCGGCCGGGCG
                                                              CTTCCGCCCCGCCCGCCGACTGGGCCGAC
~~~

# Switches

Other useful switches include (see `quorum --help` for a short
description of all of them).

* `--threads NUMBER`

Number of threads to use.

* `--kmer-len LENGTH`

Length of k-mer to use. Defaults to 24. This is limited to 31.

* `--contaminant FILE`

Pass in a fasta or fastq file of contaminant sequences. The error
correction program will truncate any reads which contains a k-mer
present in the contaminant sequences.

* `--prefix NAME`

By default, all output file have the form `quorum_*`. This can be
changed with this switch.

* `--min-q-char ASCII`

This is the ASCII value of the base of quality encoding. If not
specified, it is auto-detected: the first 1,000 reads of the first
file are read and the minimum quality value seen in these reads is
used for min-q-char. An error is raised if this auto-detected base is
not one of the standard value (33, 59 or 64).
