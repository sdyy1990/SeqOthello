# SeqOthello

__SeqOthello__ is an ultra-fast and memory-efficient indexing structure to support arbitrary sequence query against large collections of RNA-seq experiments. 



## SeqOthello Installation

### Requirements
__SeqOthello__ is tested on linux platforms with the following system settings.
 The performance is optimized for Intel CPUs with SSE4.2 support.

  * cmake >= 2.8.4
  * gcc >= 4.9.1
  * zlib >= 1.2.3

Ubuntu
```
sudo apt install cmake build-essential zlib1g-dev
```
Fedora
```
sudo yum install gcc-c++ cmake zlib-dev
```

Mac OS 10.12. Please install cmake using [brew](https://brew.sh/)
```
brew install cmake
```

### Installation

1. Clone the repository:

    ```
    git clone https://github.com/LiuBioinfo/SeqOthello.git
    ```

1. Build SeqOthello:

    ```
    cd SeqOthello/
    ./compile.sh
    ```

    After successfully build the code, you can find __SeqOthello__ toolchain at ``build/bin/``.

    ```
    ls build/bin
    ```

## Run SeqOthello

### Build SeqOthello with an example set of k-mer files.

The construction of __SeqOthello__ requires the input of a list of
k-mer files, each of which contains the set of _k_-mers extracted from
reads of the corresponding RNA-Seq experiment.
Currently the k-mer file is generated by [Jellyfish](https://github.com/gmarcais/Jellyfish) from fasta/fastq files.

For demonstration purpose, we provide ``createToy.sh`` to create a toy set of 182 experiments and all the necessary scripts to build the __SeqOthello__ structure.

1. Create example set of k-mers and the script to construct SeqOthello.
   
    After compiling, execute ``createToy.sh`` in the source folder.

    ```./createToy.sh```

    The following folder/files will be generated. You may run ``tree -d example/`` to list the generated files.

    ```
    example
      |-- bin     # folder for binary files
      |-- grp     # folder for group files
      |-- kmer    # kmer files, F0.Kmer, F1.Kmer ...
      `-- out     # SeqOthello structure
    ```

    You will find 3 scripts in the ``example/`` folder. We will use them to
    construct __SeqOthello__.

    ```
    ConvertToBinary.sh
    MakeGroup.sh
    BuildSeqOthello.sh
    ```

1. Make __SeqOthello__ binary kmer files

    ``ConvertToBinary.sh`` will convert all _k_-mer files listed in ``example/kmer/flist`` to the binary format and store them in ``example/bin/``. In this example, all kmers will be
    used to build __SeqOthello__. In practice, you can further filter the kmers
     by setting a threshold using _k_-mer counts. _k_-mers with counts below the threshold will be removed from the experiment. Please see the [manual](manual.md)
     for detailed usage.

    ```
    example/ConvertToBinary.sh
    ls example/bin/
    ```

1. Make __SeqOthello__  group files

    ``MakeGroup.sh`` obtain the _k_-mer occurrence maps within small subsets of
    experiments, where each group contains approximately 50 samples.

    ```
    example/MakeGroup.sh
    ls example/grp/
    ```

1. Build __SeqOthello__ mapping

    ``BuildSeqOthello.sh`` build __SeqOthello__ mapping between the entire set
    of _k_-mers and their experiment ids.

    ```
    example/BuildSeqOthello.sh
    ls example/map/
    ```

### Transcripts Query

__SeqOthello__ supports Containment and Coverage query modes.

1. __Containment Query__

Containment Query will return the total number of _k_-mer hits in each
experiment in a tab-delimited table. Run the code below to query the
transcript ``example/kmer/test.fa`` in the example __SeqOthello__ map.

```
build/bin/Query --map-folder=example/out/ \
--transcript=example/kmer/test.fa \
--output=example/re_containment \
--qthread=8
```

Check the query results of the first transcript (# 0) for the first 11 experiments
```
cut -f1-12 example/re_containment
```

|Transcript index | F0 | F1 | F2 | F3 | F4 | F5 | F6 | F7 | F8 | F9 | F10 |
|--|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|# 0|1211|1211|1211|1211|1211|1211|1211|1211|1211|1211|8267|

2. __Coverage Query__

Coverage query will return the detailed _k_-mer hits for each of _k_-mer
in the queried transcripts.

```
build/bin/Query --map-folder=example/out/ \
--transcript=example/kmer/test.fa \
--output=example/re_coverage \
--detail \
--qthread=8
```


The coverage result has two columns. The first column shows the _k_-mer
sequence. Column 2 use ``+`` and ``.`` signs to indicate the the _k_-mer hits status for each experiment in the
SeqOthello map. A ``+`` sign indicates the k-mer exists in the experiment, whearas ``.`` indicates otherwise.

Run ``head -n 10 example/re_coverage` to observe the first line of the coverage query result.

```
GGATAGCCCGGGTACGGACG ++++++++++++++++++++..........................................................................................................................................................++++++++++++++++++++++++++++..........................................................................................................................................................++++++++
GGGATAGCCCGGGTACGGAC ++++++++++++++++++++.........................................................................................................................................................+++++++++++++++++++++++++++++.........................................................................................................................................................+++++++++
GGGGATAGCCCGGGTACGGA ++++++++++++++++++++........................................................................................................................................................++++++++++++++++++++++++++++++........................................................................................................................................................++++++++++
CGGGGATAGCCCGGGTACGG ++++++++++++++++++++.......................................................................................................................................................+++++++++++++++++++++++++++++++.......................................................................................................................................................+++++++++++
CCGGGGATAGCCCGGGTACG ++++++++++++++++.+++......................................................................................................................................................++++++++++++++++++++++++++++.+++......................................................................................................................................................++++++++++++
TCCGGGGATAGCCCGGGTAC ++++++++++++++++.+++.....................................................................................................................................................+++++++++++++++++++++++++++++.+++.....................................................................................................................................................+++++++++++++
ATCCGGGGATAGCCCGGGTA ++++++++++++++.+.+++....................................................................................................................................................++++++++++++++++++++++++++++.+.+++....................................................................................................................................................++++++++++++++
AATCCGGGGATAGCCCGGGT ++++++++++++++.+.+++...................................................................................................................................................+++++++++++++++++++++++++++++.+.+++...................................................................................................................................................+++++++++++++++
TAATCCGGGGATAGCCCGGG ++++++++++++.+.+.+++..................................................................................................................................................++++++++++++++++++++++++++++.+.+.+++..................................................................................................................................................++++++++++++++++
GTAATCCGGGGATAGCCCGG +++++++++++++..+.+++.................................................................................................................................................++++++++++++++++++++++++++++++..+.+++.................................................................................................................................................+++++++++++++++++
```

## SeqOthello Online

__SeqOthello__ also accommodates online features for small-batch queries. Online queries preload the entire index into memory prior to querying, and can be executed in approximately 0.09 seconds per transcript.

Use the following command to start a server on the machine, (e.g., on TCP port 3322). The service will run as a deamon.

```
build/bin/Query \
--map-folder=example/out/ \
--start-server-port 3322
```

Open a new terminal, run the Client program for containment query.

```
build/bin/Client \
--transcript=kmer/test.fa \
--output=example/re_coverage_online \
--port=3322
```

## Build SeqOthello in parallel

Two of the three steps in __SeqOthello__ construction can be easily paralleled.
For example, with GNU Parallel, you can convert _k_-mer files with:

```
cat ConvertToBinary.sh | parallel
```

Then, you can make the group files with:

```
cat MakeGroup.sh | parallel
```

## License
Please refer to LICENSE.TXT.


## Getting help
For questions running __SeqOthello__, please post to [SeqOthello Google Group](https://groups.google.com/forum/#!forum/seqothello)

## Known issues

Usually, on Linux systems, there is a limit on the number of files that can be opened simutaiously. The Build program of SeqOthello may hit such limit. To edit this limit, set ulimit to larger numbers. e.g,
``ulimit -nS 4096``
