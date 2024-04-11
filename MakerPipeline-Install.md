# MAKER-pipeline

1. **Requirements**
    - Mamba, git-LFS, and git software are required

2. **Create a conda environment**
    ```bash
    $ mamba create -n <env name> snakemake singularity perl-app-cpanminus
    ```

3. **Obtain the genome annotation workflow from the Brown Lab GitHub**
    ```bash
    $ git clone https://github.com/TheBrownLab/MAKER-snakemake.git
    ```

4. **Download GeneMarkES and the GeneMark key file**
    - [Download GeneMarkES](http://topaz.gatech.edu/GeneMark/license_download.cgi)
        - Select GeneMark-ES/ET/EP+ ver 4.72_lic and the correct operating system (LINUX 64 kernel 2.6 – 4)
        - Enter your information and click the ‘Agree to terms’ button (not enter)
    ```bash
    $ wget http://topaz.gatech.edu/GeneMark/tmp/GMtool_1O1kJ/gmes_linux_64.tar.gz
    $ wget http://topaz.gatech.edu/GeneMark/tmp/GMtool_1O1kJ/gm_key_64.gz
    ```
    - Copy the gm_key to home directory
    ```bash
    $ cp <path/to/key/file> ~/.gm_key
    ```

5. **Install GeneMarkES**
    - Decompress the software folder
    ```bash
    $ tar -xf gmes_linux_64.tar
    $ cd gmes_linux_64
    ```
    - Activate your conda environment
    - Install Perl modules
    ```bash
    $ cpanm YAML Hash::Merge Parallel::ForkManager MCE::Mutex Thread::Queue threads Math::Utils
    ```
    - Check installation
    ```bash
    $ ./check_install.bash
    ```
    - If error “ ‘C’ compiler not found”
    ```bash
    $ conda install gcc_linux_64
    ```

6. **Decompress Augustus resources file**
    - Enter resources directory
    ```bash
    $ cd <path/to/MAKER-snakemake>/resources
    ```
    - Decompress file
    ```bash
    $ tar -xzvf augustus.tar.gz
    ```

## Running the pipeline:

- **Prepare data**
    - Create a new directory with MAKER-snakemake
    ```bash
    $ mkdir data
    ```
    - Copy data files to data directory
    - Assembled genome required, add *.genome.fas to end of file name when copied to data directory
    - EST data optional, add *.est.fas to end of file name when copied
    - Protein Homology data optional, add *.protein.fas to end of file name when copied

- **Perform a dry run of pipeline**
    ```bash
    $ python3 scripts/run_maker_pipeline.py <sample_name> <path.to.gmes_petap.pl>
    ```

- **Run workflow**
    - A submission script is required
    - Include the conda activation for your environment
    - Include exporting the path to perl5 and gme_linux_64
    ```bash
    #!/bin/bash
    #SBATCH -o job-%x.%j.out
    #SBATCH --cpus-per-task <number of threads>

    eval "$(conda shell.bash hook)"
    conda activate <env_name>

    export PERL5LIB=<path/to/perl5>
    export PATH="<path/to/gmes_linux_64>:$PATH"

    python3 -u scripts/run_maker_pipeline.py <sample> <path/to/gmes_petap.pl -t <threads> -x 
    ```
- **May need to run from command line first to establish conda environments within snakemake file**
- **If no RNAseq data, changes within the snakemake file are required**
