Bootstrap: docker

From: continuumio/miniconda3

%files
    ncbi-blast-2.12.0+ /share/ncbi-blast-2.12.0+
    ncbi-igblast-1.17.1 /share/ncbi-igblast-1.17.1
    database /share/database
    environment_dev.yml /environment_dev.yml
    dandelion_preprocess.py /share/dandelion_preprocess.py
    sources.list /etc/apt/sources.list
    tests /tests

%post
    apt update -y --allow-insecure-repositories && apt install -y --allow-unauthenticated sudo software-properties-common gnupg
    sudo apt -y autoremove
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 && \
    sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/' && \
    sudo apt update -y && sudo apt install -y curl libcurl4-openssl-dev rsync r-base r-base-core r-recommended r-base-dev vim
    
    # install R packages
    R --slave -e 'install.packages("remotes", repos="https://cloud.r-project.org/")'
    sysreqs=$(R --slave -e 'cat("apt-get update -y && apt-get install -y", paste(gsub("apt-get install -y ", "", remotes::system_requirements("ubuntu", "20.04", package = c("shazam","alakazam","tigger","airr","optparse","Biostrings","GenomicAlignments","IRanges","BiocManager","RCurl","XML"))), collapse = " "))')
    echo $sysreqs
    sudo -s eval "$sysreqs"
    sudo apt-get autoremove -y

    # Install required R packages
    R --slave -e 'install.packages("BiocManager", repos="https://cloud.r-project.org/")'
    R --slave -e 'BiocManager::install(c("Biostrings", "GenomicAlignments", "IRanges"))'
    R --slave -e 'install.packages(c("optparse", "alakazam", "tigger", "airr", "shazam"), repos="https://cloud.r-project.org/")'
    . /opt/conda/etc/profile.d/conda.sh
    conda env update --name sc-dandelion-container -f environment_dev.yml
    
    # fix pathing
    echo "export GERMLINE=/share/database/germlines/" | tee -a $SINGULARITY_ENVIRONMENT
    echo "export IGDATA=/share/database/igblast/" | tee -a $SINGULARITY_ENVIRONMENT
    echo "export BLASTDB=/share/database/blast/" | tee -a $SINGULARITY_ENVIRONMENT
    echo "export PATH=/share/ncbi-blast-2.12.0+/bin:/share/ncbi-igblast-1.17.1/bin:$PATH" | tee -a $SINGULARITY_ENVIRONMENT
    echo ". /opt/conda/etc/profile.d/conda.sh" | tee -a $SINGULARITY_ENVIRONMENT
    echo "conda activate sc-dandelion-container" | tee -a $SINGULARITY_ENVIRONMENT

%environment
    export GERMLINE=/share/database/germlines/
    export IGDATA=/share/database/igblast/
    export BLASTDB=/share/database/blast/
    export PATH=/share/ncbi-blast-2.12.0+/bin:/share/ncbi-igblast-1.17.1/bin:$PATH

%runscript
    alias dandelion-preprocess='/share/dandelion_preprocess.py'
    eval ${@}

%test
    ls /share
    which blastn
    conda list
    pytest -p no:cacheprovider /tests -W ignore::DeprecationWarning -W ignore::PendingDeprecationWarning -W ignore::FutureWarning

