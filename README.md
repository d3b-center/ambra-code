# Ambra code
The included tools are intended for interfacing with the Ambra Health platform for imaging (radiology) studies, utilizing the Ambra SDK which calls the Ambra API. Requires the user to have the appropriate permissions on Ambra to perform the given actions.

## Overview

This repository contains small operational scripts for:
- downloading studies from Ambra
- counting studies and subjects in an Ambra namespace
- updating Ambra study metadata by converting tags into custom field values

The scripts are written for direct use in a configured Ambra environment rather than as a generalized package.

## Installation

Install the Python dependencies:

```bash
pip install -r requirements.txt
```

Current dependency list:
- `ambra-sdk`

## Repository Contents

- `ambra_download.py`: download one study, all studies for an MRN, or all studies in a namespace
- `QC_studies.py`: count unique subjects and studies in a namespace
- `switch_tags_2_fields.py`: migrate Ambra study tags into custom field values and remove the original tags
- `exceptions.py`: local exception class used by the scripts

### Code

The ```ambra_download``` command can be used to download data from Ambra. Based on user input it will download all studies in a given location or a specific study or studies.

For quick review, ```QC_studies.py``` will print the number of unique studies and subjects found for a given bucket.

The `switch_tags_2_fields.py` script can be used to update studies in D3b namespaces by mapping known Ambra tags to a custom field and then deleting the original tags.

### Variables
Expected environment variables:

```
AMBRA_USERNAME
      required, user's associated e-mail for Ambra account

AMBRA_PASSWORD
      required, user's associated password for Ambra account
```

For QC_studies.py only:
```
AMBRA_PHI_NAMESPACE
      required, the PHI namespace ID for the given Ambra bucket/project
```

Notes on environment and endpoints:

- `ambra_download.py` and `QC_studies.py` are configured to use `https://access.dicomgrid.com/api/v3/`
- `switch_tags_2_fields.py` is configured to use `https://choparcus.ambrahealth.com/api/v3/`
- Review the endpoint in each script before running it in a different Ambra environment

### Inputs for ambra_download
```
download_type
      required, specifies the type of download. Either 'all' (all studies on the instance), 'mrn' (all studies associated with a given patient), or 'accession_number' (one study based on an accession number).

patient_mrn
      required if download_type is 'mrn', the patient MRN to download studies for.

accession_number
      required if download_type is 'accession_number', the accession number of the study to download.

ambra_phi_namespace
      required if download_type is 'all', the Ambra PHI Namepsace defining the source bucket.    
```

Behavior notes for `ambra_download.py`:

- Downloads are requested as DICOM zip bundles
- Output files are written into the current working directory
- If a zip file with the same accession number already exists, the script appends `_1`, `_2`, and so on until it finds a unique file name

### Example usage
```
python3 ambra_download.py --download_type mrn --patient_mrn 012345
python3 ambra_download.py --download_type all --ambra_phi_namespace '01234a56b-1234-0aaa-ab01-0ab12c34d567'
```

Example usage for `QC_studies.py`:

```bash
export AMBRA_USERNAME="your_ambra_username"
export AMBRA_PASSWORD="your_ambra_password"
export AMBRA_PHI_NAMESPACE="your_phi_namespace"
python3 QC_studies.py
```

What `QC_studies.py` does:

- connects to Ambra using the credentials in the environment
- queries all studies in the configured PHI namespace
- prints the number of unique patient IDs and the total number of studies found

What `switch_tags_2_fields.py` does:

- authenticates to the CHOP Arcus Ambra endpoint
- iterates through namespaces accessible to the current user
- filters to namespaces with `D3b` in the name
- finds studies with tags such as `processed_Flywheel` and `error`
- writes corresponding values into a hard-coded custom field UUID
- deletes the original tags after the custom field is updated

Operational notes for `switch_tags_2_fields.py`:

- the custom field UUID is hard-coded in the script
- the script performs write operations against Ambra and is not read-only
- the script assumes the executing user has permission to modify studies in the target namespaces

### Other resources

Ambra-SDK documentation: https://dicomgrid.github.io/sdk-python/index.html

Ambra API documentation: https://access.dicomgrid.com/api/v3/api.html
