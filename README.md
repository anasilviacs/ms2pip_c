# MS²PIP
MS²PIP is a tool to predict MS² signal peak intensities from peptide sequences.
It employs the XGBoost machine learning algorithm and is written in Python.

You can install MS²PIP on your machine by following the [instructions below](https://github.com/compomics/ms2pip_c#installation) or the [extended install instructions](https://github.com/compomics/ms2pip_c/wiki/Extended_install_instructions).
For a more user friendly experience, we created a [web server](https://iomics.ugent.be/ms2pip)
. There, you can easily upload a list of peptide sequences, after which the
corresponding predicted MS² spectra can be downloaded in a CSV or MGF file
format. The web server can also be contacted through the
[REST API](https://iomics.ugent.be/ms2pip/api/).

If you use MS²PIP for your research, please cite the following papers:
- Degroeve, S., Maddelein, D., & Martens, L. (2015). MS²PIP prediction server:
compute and visualize MS² peak intensity predictions for CID and HCD
fragmentation. Nucleic Acids Research, 43(W1), W326–W330.
https://doi.org/10.1093/nar/gkv542
- Degroeve, S., & Martens, L. (2013). MS²PIP: a tool for MS/MS peak intensity
prediction. Bioinformatics (Oxford, England), 29(24), 3199–203.
https://doi.org/10.1093/bioinformatics/btt544

## Installation
MS2PIPc runs on Python 3.5 or greater. The required Python packages are listed
in `requirements.txt`. MS2PIPc also requires machine specific compilation of the
C-code:
```
sh compile.sh
```
Check out the [extended install instructions](https://github.com/compomics/ms2pip_c/wiki/Extended_install_instructions)
for a more detailed explanation.


## Predicting MS2 peak intensities
MS2PIPc comes with pre-trained models for a variety of fragmentation methods and
modifications. These models can easily be applied by configuring MS2PIPc in the
[config.txt file](https://github.com/compomics/ms2pip_c#config-file) and
providing a list of peptides in the form of a [PEPREC file](https://github.com/compomics/ms2pip_c#peprec-file).

### MS2PIPc command line interface
```
usage: ms2pipC.py [-h] [-c FILE] [-s FILE] [-w FILE] [-m INT] <peptide file>

positional arguments:
  <peptide file>  list of peptides

optional arguments:
  -h, --help      show this help message and exit
  -c FILE         config file (by default config.txt)
  -s FILE         .mgf MS2 spectrum file (optional)
  -w FILE         write feature vectors to FILE.{pkl,h5} (optional)
  -m INT          number of cpu's to use
```

### Config file
Several MS2PIPc options need to be set in this config file.

The models that should be used are set as `frag_method=X` where X is
either `CID`, `HCD`, `HCDch2`, `ETD`, `HCDiTRAQ4` or
`HCDiTRAQ4phospho`. If the `frag_method` is set to `HCDch2`, MS2PIP
will predict intensities for HCD charge +1 and charge 2+ fragment ions.

The fragment ion error tolerance is set as `frag_error=X` where is X is
the tolerance in Da.

PTMs (see further) are set as `ptm=X,Y,opt,Z` for each internal PTM
where X is a string that represents the PTM, Y is the difference in Da
associated with the PTM, opt is a required for compatibility with
other CompOmics projects, and Z is the amino acid that is modified by the PTM.
For N- and C-terminal modifications, Z should be `N-term` or `C-term`,
respectively.


### PEPREC file
To apply the pre-trained models you need to pass *only* a `<peptide file>`
to `ms2pipC.py`. This file contains the peptide sequences for which you
want to predict the b- and y-ion peak intensities. The file is space
separated and contains four columns with the following header names:

- `spec_id`: an id for the peptide/spectrum
- `modifications`: a string indicating the modified amino acids
- `peptide`: the unmodified amino acid sequence
- `charge`: charge state to predict

The *spec_id* column is a unique identifier for each peptide that will
be used in the TITLE field of the predicted MS2 `.mgf` file. The
`modifications` column is a string that lists the PTMs in the peptide.
Each PTM is written as `A|B` where A is the location of the PTM in the
peptide (the first amino acid has location 1, location 0 is used for
n-term modifications, while -1 is used for c-term modifications) and B
is a string that represent the PTM as defined in the config file (`-c`
command line argument). Multiple PTMs in the `modifications` column are
concatenated with '|'.

As an example, suppose the config file contains the line
```
ptm=Cam,57.02146,opt,C
ptm=Ace,42.010565,opt,N-term
ptm=Glyloss,-58.005479,opt,C-term
```
then a modifications string could like `0|Ace|2|Cam|5|Cam|-1|Glyloss`
which means that the second and fifth amino acid is modified with `Cam`,
that there is an N-terminal modification `Ace`, and that there is a
C-terminal modification `Glyloss`.

The predictions are saved in a `.csv` file with the name
`<peptide_file>_predictions.csv`.
If you want the output to be in the form of an `.mgf` file, replace the
variable `mgf` in line 716 of `ms2pipC.py`.

To train custom MS2PIPc models, please refer to [Training new MS2PIP models](https://github.com/compomics/ms2pip_c/wiki/Training_new_MS2PIP_models) on our Wiki pages.
