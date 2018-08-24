# Summary Statistics harmonisation


Harmonisation scripts to run the processes outlined below.

The pipeline is managed by `snakemake`, so it can be followed in the [Snakefile](Snakefile).

Most of the harmonisation is performed by [sumstat_harmoniser](https://github.com/opentargets/sumstat_harmoniser) which is used to a) find the orientation of the variants, b) resolve RSIDs from locations and alleles and c) orientate the variants to the reference strand.

# Running the pipeline

The following are required:

python3
[HTSlib](http://www.htslib.org/download/)

python libraries:
[snakemake](https://snakemake.readthedocs.io/en/stable/getting_started/installation.html)
h5py
requests

Follow the Ensembl Perl API installation instructions [here](https://www.ensembl.org/info/docs/api/api_installation.html).
If you don't have access to a mirror of the Ensembl Homo sapiens core and variation databases, you will need to [build your own](https://www.ensembl.org/info/docs/webcode/mirror/install/ensembl-data.html). If it is not possible to connect to a mirror, the RSID --> location mapping will all be done using liftover, so if the location data are not present in your file, those records will be lost.
Once you have your mirror of the Ensembl Homo sapiens core and variation databases you need to update the [registry file](https://github.com/EBISPOT/sum-stats-formatter/blob/master/harmonisation/formatting_tools/ensembl.registry) with the appropriate information.

To run the pipeline simply run `snakemake` but be warned that this is not very optimal.
To run on the cluster, ssh in, clone this repository and submodules and install dependencies (above) and Ensembl data.
Run `snakemake -j {number of jobs} -w {wait time} --cluster 'bsub {options}' &` to run it on the background. 
To run 100 jobs in parallel do the following for example:
`snakemake -j 100 -w 1800 --cluster 'bsub -o stdin.txt -e stderr' &`
The wait time limit of 1800 seconds should provide enough time to wait for output files to be generated but it should be extended/shortened as required.

```
#pseudocode

format for sumstats database

FOR each variant in file
    IF RSID maps to genomic location in Ensembl THEN
        Update locations based on Ensembl mapping
    ELIF can liftover locations to current build THEN
        liftover locations to current build
    ELSE
        Remove variant
    ENDIF
ENDFOR

FOR each non-palindromic variant
    check orientation (query Ensembl reference VCF with chr:bp, effect and other alleles)
ENDFOR

summarise the orientation of the variants: outcomes are ‘forward’, ‘reverse’ or ‘mixed’

IF ‘mixed’ THEN
    remove palindromic variants
ELSE
    proceed with all variants (including palindromic snps) assuming consensus orientation
ENDIF

FOR each remaining variant:
    get rsid and update variant_id
    check orientation of variant against Ensembl reference VCF
    orientate variant to reference (can flip alleles, betas, ORs, CIs allele frequencies)
    remove if variant_id, p_value, base_pair_location, chromosome are invalid
ENDFOR
```