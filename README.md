# GenoML 

<p align="center">
  <img width="300" height="300" src="https://github.com/GenoML/genoml2/blob/master/logo.png">
</p>

[![Downloads](https://static.pepy.tech/personalized-badge/genoml2?period=total&units=international_system&left_color=black&right_color=grey&left_text=Downloads)](https://pepy.tech/project/genoml2)

> Updated 2 March 2021: Latest Release on pip! v1.0.0.beta.11

# How to Get Started with GenoML


### Introduction
[GenoML (**Geno**mics + **M**achine **L**earning)](https://genoml.com) is an automated Machine Learning (autoML) for genomics data. In general, use a Linux or Mac with Python >3.5 for best results. **This [repository](https://github.com/GenoML/genoml2) and [pip package](https://pypi.org/project/genoml2/) are under active development!** 

This README is a brief look into how to structure arguments and what arguments are available at each phase for the GenoML CLI. 

If you are using GenoML for your own work, please cite the following papers: 
- Makarious, M. B., Leonard, H. L., Vitale, D., Iwaki, H., Saffo, D., Sargent, L., ... & Faghri, F. (2021). GenoML: Automated Machine Learning for Genomics. arXiv preprint arXiv:2103.03221
- Makarious, M. B., Leonard, H. L., Vitale, D., Iwaki, H., Sargent, L., Dadu, A., ... & Nalls, M. A. (2021). Multi-Modality Machine Learning Predicting Parkinson’s Disease. bioRxiv.

### Installing + Downloading Example Data 
- Install this repository directly from GitHub (from source; master branch)

`git clone https://github.com/GenoML/genoml2.git`

- Install using pip or upgrade using pip

`pip install genoml2`

OR

`pip install genoml2 --upgrade`

- To install the `examples/` directory (~315 KB), you can use SVN (pre-installed on most Macs)

`svn export https://github.com/GenoML/genoml2.git/trunk/examples`

> Note: When you pip install this package, the examples/ folder is also downloaded! However, if  you still want to download the directory and SVN is not pre-installed, you can download it via Homebrew if you have that installed using `brew install svn` 

### Table of Contents 
#### [0. (OPTIONAL) How to Set Up a Virtual Environment via Conda](#0)
#### [1. Munging with GenoML](#1)
#### [2. Training with GenoML](#2)
#### [3. Tuning with GenoML](#3)
#### [4. Testing/Validating with GenoML](#4)
#### [5. Experimental Features](#5)

<a id="0"></a>
## 0. [OPTIONAL] How to Set Up a Virtual Environment via Conda

You can create a virtual environment to run GenoML, if you prefer.
If you already have the [Anaconda Distribution](https://www.anaconda.com/products/individual#download), this is fairly simple.

To create and activate a virtual environment:

```shell
# To create a virtual environment
conda create -n GenoML python=3.7

# To activate a virtual environment
conda activate GenoML

# To install requirements via pip 
pip install -r requirements.txt
    # If issues installing xgboost from requirements - (3 options)
        # use Homebrew to 
            # xcode-select --install
            # brew install gcc@7
        # conda install -c conda-forge xgboost 
        # pip install xgboost==0.90
    # If issues installing umap 
        # pip install umap-learn 

## MISC
# To deactivate the virtual environment
# conda deactivate GenoML

# To delete your virtual environment
# conda env remove -n GenoML
```

To install the GenoML in the user's path in a virtual environment, you can do the following:
```shell
# Install the package at this path
pip install .

# MISC
	# To save out the environment requirements to a .txt file
# pip freeze > requirements.txt

	# Removing a conda virtualenv
# conda remove --name GenoML --all 
```

<a id="1"></a>
## 1. Munging with GenoML

Munging with GenoML will, at minimum, do the following: 
- Prune your genotypes using PLINK v1.9 (if `--geno` flag is used)
- Impute per column using median or mean (can be changed with the `--impute` flag)
- Z-scaling of features and removing columns with a std dev = 0 

**Required** arguments for GenoML munging are `--prefix` and `--pheno` 
- `data` : Is the data `continuous` or `discrete`?
- `method`: Do you want to use `supervised` or `unsupervised` machine learning? *(unsupervised currently under development)*
- `mode`:  would you like to `munge`, `train`, `tune`, or `test` your model?
- `--prefix` : Where would you like your outputs to be saved?
- `--pheno` : Where is your phenotype file? This file only has 2 columns, ID in one, and PHENO in the other (0 for controls and 1 for cases)


Be sure to have your files formatted the same as the examples, key points being: 
- **0=controls and 1=case** in your phenotype file
- Your phenotype file consisting **only** of the "ID" and "PHENO" columns
- Your sample IDs matching across all files
- Your sample IDs not consisting with only integers (add a prefix or suffix to all sample IDs ensuring they are alphanumeric if this is the case prior to running GenoML)
- Please avoid the use of characters like commas, semi-colons, etc. in the column headers (it is Python after all!)  

> *Note:* The following examples are for discrete data, but if you substitute following commands with `continuous` instead of discrete, you can preprocess your continuous data!

If you would like to munge just with genotypes (in PLINK binary format), the simplest command is the following: 
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files and a phenotype file 

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--pheno examples/discrete/training_pheno.csv
```

If you would like to control the pruning stringency in genotypes: 
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files and a phenotype file 

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--r2_cutoff 0.3 \
--pheno examples/discrete/training_pheno.csv
```

You can choose to skip pruning your SNPs at this stage by changing the `--skip_prune` flag to "yes" (default is "no")
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files and a phenotype file 

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--skip_prune yes \
--pheno examples/discrete/training_pheno.csv
```

You can choose to impute on `mean` or `median` by modifying the `--impute` flag, like so *(default is median)*:
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files and a phenotype file and specifying impute

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--pheno examples/discrete/training_pheno.csv \
--impute mean
```

If you suspect collinear variables, and think this will be a problem for training the model moving forward, you can use [variance inflation factor](https://www.investopedia.com/terms/v/variance-inflation-factor.asp) (VIF) filtering: 
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files and a phenotype file while using VIF to remove multicollinearity 

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--pheno examples/discrete/training_pheno.csv \
--vif 5 \
--iter 1
```

- The `--vif` flag specifies the VIF threshold you would like to use (5 is recommended) 
- The number of iterations you'd like to run can be modified with the `--iter` flag (if you have or anticipate many collinear variables, it's a good idea to increase the iterations)

Well, what if you had GWAS summary statistics handy, and would like to just use the same SNPs outlined in that file? You can do so by running the following:
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files, a phenotype file, and a GWAS summary statistics file 

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--pheno examples/discrete/training_pheno.csv \
--gwas examples/discrete/example_GWAS.csv
```
> *Note:* When using the GWAS flag, the PLINK binaries will be pruned to include matching SNPs to the GWAS file. 

...and if you wanted to add a p-value cut-off...
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files, a phenotype file, and a GWAS summary statistics file with a p-value cut-off 

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--pheno examples/discrete/training_pheno.csv \
--gwas examples/discrete/example_GWAS.csv \
--p 0.01
```
Do you have additional data you would like to incorporate? Perhaps clinical, demographic, or transcriptomics data? If coded and all numerical, these can be added as an `--addit` file by doing the following: 
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files, a phenotype file, and an addit file

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--pheno examples/discrete/training_pheno.csv \
--addit examples/discrete/training_addit.csv
```
You also have the option of not using PLINK binary files if you would like to just preprocess (and then, later train) on a phenotype and addit file by doing the following:
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files, a phenotype file, and an addit file

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--pheno examples/discrete/training_pheno.csv \
--addit examples/discrete/training_addit.csv
```

Are you interested in selecting and ranking your features? If so, you can use the `--feature_selection` flag and specify like so...:
```shell
# Running GenoML munging on discrete data using PLINK genotype binary files, a phenotype file, and running feature selection 

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--pheno examples/discrete/training_pheno.csv \
--addit examples/discrete/training_addit.csv \
--feature_selection 50
```
The `--feature_selection` flag uses extraTrees ([classifier](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.ExtraTreesClassifier.html) for discrete data; [regressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.ExtraTreesRegressor.html) for continuous data) to output a `*.approx_feature_importance.txt` file with the features most contributing to your model at the top. 

Do you have additional covariates and confounders you would like to adjust for in the munging step prior to training your model and/or would like to reduce your data? To adjust, use the `--adjust_data` flag with the following necessary flags: 
- `--adjust_normalize`: Would you like to normalize your final adjusted data? (Default: yes)
- `--target_features`: A .txt file, one column, with a list of features to adjust (no header). These should correspond to features in the munged dataset
- `--confounders`: A .csv of confounders to adjust for with ID column and header. Numeric, with no missing data and the ID column is mandatory (this can be PCs, for example)

To reduce your data prior to adjusting, use the `--umap_reduce yes` flag. This flag will also prompt you for if you want to adjust your data, normalize, and what your target features and confounders might be. We use the [Uniform Manifold Approximation and Projection for Dimension Reduction](https://umap-learn.readthedocs.io/en/latest/) (UMAP) to reduce your data into 2D, adjust, and exports a plot and an adjusted dataframe moving forward. This can be done by running the following: 

```shell
# Running GenoML munging on discreate data using PLINK binary files, a phenotype file, using UMAP to reduce dimensions and account for features, and running feature selection

genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--pheno examples/discrete/training_pheno.csv \
--addit examples/discrete/training_addit.csv \
--umap_reduce yes \
--adjust_data yes \
--adjust_normalize yes \
--target_features examples/discrete/to_adjust.txt \
--confounders examples/discrete/training_addit_confounder_example.csv \
--feature_selection 50 
```
Here, the `--confounders` flag takes in a dataset of features that should be accounted for. This is a .csv file with the ID column and header included and is numeric with no missing data. **The ID column is mandatory.** The `--target_features` flag takes in a .txt with a list of features (column names) you are adjusting for.

<a id="2"></a>
## 2. Training with GenoML
Training with GenoML competes a number of different algorithms and outputs the best algorithm based on a specific metric that can be tweaked using the `--metric_max` flag *(default is AUC)*.

**Required** arguments for GenoML are the following: 
- `data` : Is the data `continuous` or `discrete`?
- `method`: Do you want to use `supervised` or `unsupervised` machine learning? *(unsupervised currently under development)*
- `mode`:  would you like to `munge`, `train`, `tune`, or `test` your model?
- `--prefix` : Where would you like your outputs to be saved?

The most basic command to train your model looks like the following, it looks for the `*.dataForML` file that was generated in the munging step: 
```shell
# Running GenoML supervised training after munging on discrete data

genoml discrete supervised train \
--prefix outputs/test_discrete_geno
```

If you would like to determine the best competing algorithm by something other than the AUC, you can do so by changing the `--metric_max` flag (Options include `AUC`, `Balanced_Accuracy`, `Sensitivity`, and `Specificity`) :
```shell
# Running GenoML supervised training after munging on discrete data and specifying the metric to maximize by 

genoml discrete supervised train \
--prefix outputs/test_discrete_geno \
--metric_max Sensitivity
```
> *Note:* The `--metric_max` flag is only available for discrete datasets.

<a id="3"></a>
## 3. Tuning with GenoML

The most basic command to tune your model looks like the following, it looks for the file that was generated in the training step: 
```shell
# Running GenoML supervised tuning after munging and training on discrete data

genoml discrete supervised tune \
--prefix outputs/test_discrete_geno
```

If you are interested in changing the number of iterations the tuning process goes through by modifying `--max_tune` *(default is 50)*, or the number of cross-validations by modifying `--n_cv` *(default is 5)*, this is what the command would look like: 
```shell
# Running GenoML supervised tuning after munging and training on discrete data, modifying the number of iterations and cross-validations 

genoml discrete supervised tune \
--prefix outputs/test_discrete_geno \
--max_tune 10 --n_cv 3
```

If you are interested in tuning on another metric other than AUC *(default is AUC)*, you can modify `--metric_tune` (options are `AUC` or `Balanced_Accuracy`) by doing the following: 
```shell
# Running GenoML supervised tuning after munging and training on discrete data, modifying the metric to tune by

genoml discrete supervised tune \
--prefix outputs/test_discrete_geno \
--metric_tune Balanced_Accuracy
```

<a id="4"></a>
## 4. Testing/Validation with GenoML

In order to properly test how your model performs on a dataset it's never seen before (but you start with different PLINK binaries), we have created the harmonization step that will:
1. Keep only the same SNPs between your reference dataset and the dataset you are using for validation
2. Force the reference alleles in the validation dataset to match your reference dataset
3. Export a `.txt` file with the column names from your reference dataset to later use in the munging of your validation dataset 

Using GenoML for both your reference dataset and then your validation dataset, the process will look like the following: 

1. Munge and train your first dataset
    	- That will be your “reference” model
2. Use the outputs of step 1's munge for your reference model to harmonize your incoming validation dataset
3.  Run through harmonization step with your validation dataset
4.  Run through munging with your newly harmonized dataset
5.  Retrain your reference model with only the matching columns of your unseen data 
	- Given the nature of ML algorithms, you cannot test a model on a set of data that does not have identical features
6. Test your newly retrained reference model on the unseen data

### Harmonizing your Validation/Test Dataset 

**Required** arguments for harmonizing with GenoML are the following: 
- `--test_geno_prefix` : What is the prefix of your validation dataset PLINK binaries?
- `--test_prefix`: What do you want the output to be named?
- `--ref_model_prefix`:  What is the name of the previously GenoML-munged dataset you would like to use as your reference dataset? (Without the `.dataForML.h5` suffix)
- `--training_snps_alleles` : What are the SNPs and alleles you would like to use? (This is generated at the end of your previously-GenoML munged dataset with the suffix `variants_and_alleles.tab`)

To harmonize your incoming validation dataset to match the SNPs and alleles to your reference dataset, the command would look like the following:

```shell
# Running GenoML harmonize

genoml harmonize \
--test_geno_prefix examples/discrete/validation \
--test_prefix outputs/validation_test_discrete_geno \
--ref_model_prefix outputs/test_discrete_geno \
--training_snps_alleles outputs/test_discrete_geno.variants_and_alleles.tab
```

This step will generate: 
- a `*_refColsHarmonize_toKeep.txt` file of columns to keep for the next step 
- `*_refSNPs_andAlleles.*` PLINK binary files (.bed, .bim, and .fam) that have the SNPs and alleles match your reference dataset

Now that you have harmonized your validation dataset to your reference dataset, you can now munge using a command similar to the following:
```shell
# Running GenoML munge after GenoML harmonize

genoml discrete supervised munge --prefix outputs/validation_test_discrete_geno \
--geno outputs/validation_test_discrete_geno_refSNPs_andAlleles \
--pheno examples/discrete/validation_pheno.csv \
--addit examples/discrete/validation_addit.csv \
--ref_cols_harmonize outputs/validation_test_discrete_geno_refColsHarmonize_toKeep.txt
```
All munging options discussed above are available at this step, the only difference here is you will add the `--ref_cols_harmonize` flag to include the `*.refColsHarmonize_toKeep.txt` file generated at the end of harmonizing to only keep the same columns that the reference dataset had. 

After munging and training your reference model and harmonizing and munging your unseen test data, **you will retrain your reference model to include only matching features**. Given the nature of ML algorithms, you cannot test a model on a set of data that does not have identical features. 

To retrain your model appropriately, after munging your test data with the `--ref_cols_harmonize ` flag, a final columns list will be generated at `*.finalHarmonizedCols_toKeep.txt`. This includes all the features that match between your unseen test data and your reference model. Use the `--matching_columns` flag when retraining your reference model to use the appropriate features.

When retraining of the reference model is complete, you are ready to test!

A step-by-step guide on how to achieve this is listed below:
```shell
# 0. MUNGE THE REFERENCE DATASET
genoml discrete supervised munge \
--prefix outputs/test_discrete_geno \
--geno examples/discrete/training \
--pheno examples/discrete/training_pheno.csv
# Files made: 
    # outputs/test_discrete_geno.dataForML.h5
    # outputs/test_discrete_geno.list_features.txt
    # outputs/test_discrete_geno.variants_and_alleles.tab

# 1. TRAIN THE REFERENCE DATASET
genoml discrete supervised train \
--prefix outputs/test_discrete_geno
# Files made: 
    # outputs/test_discrete_geno.best_algorithm.txt
    # outputs/test_discrete_geno.trainedModel.joblib
    # outputs/test_discrete_geno.trainedModel_trainingSample_Predictions.csv
    # outputs/test_discrete_geno.trainedModel_withheldSample_Predictions.csv
    # outputs/test_discrete_geno.trainedModel_withheldSample_ROC.png
    # outputs/test_discrete_geno.trainedModel_withheldSample_probabilities.png
    # outputs/test_discrete_geno.training_withheldSamples_performanceMetrics.csv

# 2. HARMONIZE TEST DATASET IF USING PLINK/GENOTYPES
genoml harmonize \
--test_geno_prefix examples/discrete/validation \
--test_prefix outputs/validation_test_discrete_geno \
--ref_model_prefix outputs/test_discrete_geno \
--training_snps_alleles outputs/test_discrete_geno.variants_and_alleles.tab
# Files made: 
    # outputs/validation_test_discrete_geno.refColsHarmonize_toKeep.txt
    # outputs/validation_test_discrete_geno.refSNPs_andAlleles.bed
    # outputs/validation_test_discrete_geno.refSNPs_andAlleles.bim
    # outputs/validation_test_discrete_geno.refSNPs_andAlleles.fam

# 3. MUNGE THE TEST DATASET ON REFERENCE MODEL COLUMNS
genoml discrete supervised munge \
--prefix outputs/validation_test_discrete_geno \
--geno outputs/validation_test_discrete_geno.refSNPs_andAlleles \
--pheno examples/discrete/validation_pheno.csv \
--addit examples/discrete/validation_addit.csv \
--ref_cols_harmonize outputs/validation_test_discrete_geno.refColsHarmonize_toKeep.txt
# Files made: 
    # outputs/validation_test_discrete_geno.finalHarmonizedCols_toKeep.txt
    # outputs/validation_test_discrete_geno.list_features.txt
    # outputs/test_discrete_geno.variants_and_alleles.tab
    # outputs/validation_test_discrete_geno.dataForML.h5

# 4. RETRAIN REFERENCE MODEL ON INTERSECTING COLUMNS BETWEEN REFERENCE AND TEST
genoml discrete supervised train \
--prefix outputs/test_discrete_geno \
--matching_columns outputs/validation_test_discrete_geno.finalHarmonizedCols_toKeep.txt
# Note: This replaces the trained model you made in step 1! 
# Files made: 
    # outputs/test_discrete_geno.best_algorithm.txt
    # outputs/test_discrete_geno.trainedModel.joblib
    # outputs/test_discrete_geno.trainedModel_trainingSample_Predictions.csv
    # outputs/test_discrete_geno.trainedModel_withheldSample_Predictions.csv
    # outputs/test_discrete_geno.trainedModel_withheldSample_ROC.png
    # outputs/test_discrete_geno.trainedModel_withheldSample_probabilities.png
    # outputs/test_discrete_geno.training_withheldSamples_performanceMetrics.csv

# OPTIONAL: TUNING YOUR RETRAINED REFERENCE MODEL ON INTERSECTING COLUMNS BETWEEN REFERENCE AND TEST
genoml discrete supervised tune \
--prefix outputs/test_discrete_geno \
--matching_columns outputs/validation_test_discrete_geno.finalHarmonizedCols_toKeep.txt

# 5. TEST RETRAINED REFERENCE MODEL OR TUNED MODEL ON UNSEEN DATA
genoml discrete supervised test \
--prefix outputs/validation_test_discrete_geno \
--test_prefix outputs/validation_test_discrete_geno \
--ref_model_prefix outputs/test_discrete_geno.trainedModel
    # If testing a tuned model, change suffix from .trainedModel to .tunedModel
# Files made: 
    # outputs/validation_test_discrete_geno.testedModel_allSample_predictions.csv
    # outputs/validation_test_discrete_geno.testedModel_allSample_probabilities.png
    # outputs/validation_test_discrete_geno.testedModel_allSample_ROC.png
    # outputs/validation_test_discrete_geno.testedModel_allSamples_performanceMetrics.csv
```

> *Note:* When munging the test dataset on the reference model columns using the --ref_cols_harmonize, be sure not to include the --feature_selection flag, as you have already specified the columns to keep moving forward.

<a id="5"></a>
## 5. Experimental Features
**UNDER ACTIVE DEVELOPMENT** 

Planned experimental features include, but are not limited to:
- Unsupervised munging, training, tuning, and testing
-  GWAS QC and Pipeline
-  Network analyses
-  Meta-learning
-  Federated learning
-  Biobank-scale support
-  Cross-silo checks for genetic duplicates
-  Outlier detection
-  ...?
