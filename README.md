# Deep-Learning Seismic Catalog Workflow for the 2016 Central Italy Sequence

This repository contains a reproducible tutorial workflow for building a **preliminary earthquake catalog from continuous seismic waveform data** using deep-learning-based phase picking and probabilistic phase association.

The workflow is demonstrated using data from the **2016–2017 Amatrice–Visso–Norcia seismic sequence in Central Italy**, with a one-day example covering **30 October 2016**, when the Mw 6.5 Norcia earthquake occurred.

The tutorial performs three main processing steps:

1. retrieval of seismic waveforms and station metadata through the **ORFEUS–EIDA federated services of the EPOS Seismology community**;
2. automatic P- and S-wave picking with **PhaseNet**, using pretrained INSTANCE weights through **SeisBench**;
3. probabilistic phase association and preliminary event location with **GaMMA**.

The final output is a **raw preliminary seismic catalog** containing estimated event origin times, hypocentral coordinates and associated phase picks.

> [!IMPORTANT]
> The workflow does not calculate earthquake magnitudes and does not perform refined earthquake relocation. The resulting catalog must therefore be quality-controlled, validated and, where necessary, relocated before scientific interpretation.

This material was originally developed for a tutorial presented at the **EGU General Assembly 2025**.

---

## Table of contents

- [Purpose and learning objectives](#purpose-and-learning-objectives)
- [Workflow overview](#workflow-overview)
- [Input and output](#input-and-output)
- [EPOS Seismology services](#epos-seismology-services)
- [Case study](#case-study)
- [Methods and software](#methods-and-software)
- [Repository structure](#repository-structure)
- [Notebook description](#notebook-description)
- [Installation](#installation)
- [Configuration](#configuration)
- [How to run the tutorial](#how-to-run-the-tutorial)
- [Output files](#output-files)
- [Limitations](#limitations)
- [References](#references)
- [Contributors](#contributors)
- [License](#license)
---

## Purpose and learning objectives

The purpose of this repository is to demonstrate how continuous seismic data can be transformed into a preliminary earthquake catalog using open-source tools and standardized seismological data services.

By completing the tutorial, users will learn how to:

- define a geographical area and time interval for seismic-data retrieval;
- query federated European FDSN services;
- download continuous waveforms in MiniSEED format;
- retrieve station and channel metadata in StationXML format;
- apply a pretrained PhaseNet model to three-component seismic data;
- extract P- and S-wave picks using probability thresholds;
- associate picks recorded at different stations using GaMMA;
- obtain preliminary event origin times and hypocentral coordinates;
- export phase-pick and catalog tables;
- visualize station geometry and preliminary seismicity.

The repository is intended for educational and demonstrative use. It is not an operational earthquake-monitoring system.

---
## Workflow overview

```text
EPOS Seismology / ORFEUS–EIDA services
                  │
                  ▼
      MiniSEED seismic waveforms
      StationXML station metadata
                  │
                  ▼
       PhaseNet through SeisBench
                  │
                  ▼
        P- and S-wave pick tables
                  │
                  ▼
       GaMMA phase association
                  │
                  ▼
      Preliminary earthquake catalog
      Event-associated phase picks
```

The workflow is organized into three operational notebooks:

```text
1_download_mseed.ipynb
        │
        ▼
2_apply_picking.ipynb
        │
        ▼
3_build_the_catalog.ipynb
```

An additional introductory notebook provides a concise overview of the case study and the tutorial structure.

---
## Input and output

### Main input parameters

The user must define:

- experiment directory;
- start and end time;
- geographical bounding box;
- seismic networks;
- station selection;
- location codes;
- channel families;
- PhaseNet pretrained model;
- P- and S-pick probability thresholds;
- GaMMA association parameters;
- one-dimensional P- and S-wave velocity model;
- minimum numbers of picks required to define an event.

### Example configuration

| Parameter | Example value |
|---|---|
| Time interval | 30–31 October 2016 |
| Latitude range | 42.4°–43.2° N |
| Longitude range | 12.8°–13.6° E |
| Network selection | All available networks |
| Channel selection | `HH?`, `EH?` |
| Waveform format | MiniSEED |
| Metadata format | StationXML |
| Phase picker | PhaseNet |
| Pretrained weights | INSTANCE |
| P-pick threshold | 0.5 |
| S-pick threshold | 0.5 |
| Phase associator | GaMMA |
| Association method | Bayesian Gaussian Mixture Model |

These values are provided only as a reproducible example and can be modified.

### Main outputs

The workflow produces:

- MiniSEED waveform files;
- StationXML metadata;
- a station-coordinate table;
- waveform diagnostic plots;
- PhaseNet P- and S-wave picks;
- pick-probability values;
- station-specific pick files;
- a combined and chronologically sorted pick table;
- a preliminary earthquake catalog;
- a table linking picks to associated events;
- event coordinates in projected and geographical systems;
- catalog and station maps.

The workflow does **not** produce:

- earthquake magnitudes;
- a relocated earthquake catalog;
- focal mechanisms;
- source parameters;
- a complete uncertainty analysis.

---
## EPOS Seismology services

### Data-access model

The tutorial uses resources from the **EPOS Seismology community** through the federated **ORFEUS–EIDA** infrastructure.

The EPOS Platform can be used to discover relevant community services and datasets. During code execution, the actual waveform and station requests are sent through standardized FDSN services operated by the corresponding European data centres.

The notebook therefore does not download waveform data directly from a single central EPOS server. Instead, it uses the EIDA routing service to determine which European data centre is responsible for each requested seismic network.

### EIDA routing client

The download notebook initializes the ObsPy routing client as follows:

```python
from obspy.clients.fdsn import Client, RoutingClient

epos_eida_client = RoutingClient("eida-routing")
```

The EIDA routing service automatically forwards each request to the appropriate European FDSN node.

An optional external fallback is also configured:

```python
iris_client = Client("IRIS")

fdsn_clients = [
    ("EPOS Seismology / ORFEUS-EIDA", epos_eida_client),
    ("EarthScope/IRIS fallback", iris_client),
]
```

The external fallback is used only if the EIDA request fails. The log records which provider returned the data.
### Services used during execution

The workflow uses the following community services:

#### EIDA Routing Service

Determines the data centre responsible for the requested network, station, location, channel and time interval.

#### FDSN Station Service

Retrieves:

- network and station identifiers;
- station coordinates;
- channel information;
- sensor orientation;
- sampling information;
- instrumental-response metadata.

The metadata are stored in **StationXML** format.

#### FDSN Dataselect Service

Retrieves continuous seismic waveforms for the requested stations and channels.

The waveforms are stored in **MiniSEED** format.

### EPOS Platform references

Before publishing the repository, add the exact EPOS Platform records corresponding to the services selected for the tutorial:

- `[EPOS Platform record – EIDA routing or waveform service](ADD_EP0S_PLATFORM_URL)`
- `[EPOS Platform record – station metadata service](ADD_EP0S_PLATFORM_URL)`

The records should refer to the services actually used by the released notebook.

---

## Case study

### 2016–2017 Amatrice–Visso–Norcia seismic sequence

The tutorial uses data from the seismic sequence that affected Central Italy between 2016 and 2017.

The sequence began on **24 August 2016** with the Mw 6.0 Amatrice earthquake and continued with several major events. The largest event occurred near Norcia on **30 October 2016** and had a moment magnitude of Mw 6.5.

The example notebooks process data from 30 October 2016.

![ShakeMap of the 30 October 2016 Central Italy earthquake](https://upload.wikimedia.org/wikipedia/commons/a/a4/30-10-2016_central_italy_ShakeMap.jpg)

*ShakeMap of the 30 October 2016 Mw 6.5 Central Italy earthquake. Source: INGV via Wikimedia Commons.*

### Principal earthquakes

| Date | Local time | Magnitude | Approximate depth | Epicentral area |
|---|---:|---:|---:|---|
| 24 August 2016 | 03:36 | Mw 6.0 | 8 km | Amatrice–Accumoli |
| 24 August 2016 | 04:33 | Mw 5.3 | 8 km | Norcia |
| 26 October 2016 | 19:10 | Mw 5.4 | 9 km | Castelsantangelo sul Nera |
| 26 October 2016 | 21:18 | Mw 5.9 | 8 km | Ussita |
| 30 October 2016 | 07:40 | Mw 6.5 | 9 km | Norcia |
| 18 January 2017 | 10:25 | Mw 5.1 | 10 km | Montereale |
| 18 January 2017 | 11:14 | Mw 5.5 | 10 km | Capitignano |
| 18 January 2017 | 11:25 | Mw 5.4 | 9 km | Pizzoli |
| 18 January 2017 | 14:33 | Mw 5.0 | 10 km | Cagnano Amiterno |

This table provides contextual information and is not used directly as workflow input.

---

## Methods and software

### ObsPy

[ObsPy](https://docs.obspy.org/) is used to:

- connect to FDSN services;
- use the EIDA routing client;
- retrieve seismic waveforms;
- retrieve station metadata;
- read and write MiniSEED;
- read and write StationXML;
- manage seismic time objects;
- manipulate and visualize waveform streams.
Reference:

> Beyreuther, M., Barsch, R., Krischer, L., Megies, T., Behr, Y., and Wassermann, J. (2010). ObsPy: A Python Toolbox for Seismology. *Seismological Research Letters*, 81(3), 530–533. https://doi.org/10.1785/gssrl.81.3.530

### SeisBench

[SeisBench](https://github.com/seisbench/seisbench) provides a common framework for applying machine-learning models to seismic waveform data and distributing pretrained model weights.

In this workflow, SeisBench is used to load and apply PhaseNet.

Reference:

> Woollam, J., Münchmeyer, J., Tilmann, F., Rietbrock, A., Lange, D., Bornstein, T., et al. (2022). SeisBench—A Toolbox for Machine Learning in Seismology. *Seismological Research Letters*, 93(3), 1695–1709. https://doi.org/10.1785/0220210324

### PhaseNet

[PhaseNet](https://github.com/AI4EPS/PhaseNet) is a deep-neural-network model that estimates probabilities for:

- P-wave arrivals;
- S-wave arrivals;
- background noise.

Discrete picks are extracted where the predicted P- or S-wave probability exceeds the selected threshold.

Reference:

> Zhu, W., and Beroza, G. C. (2019). PhaseNet: A Deep-Neural-Network-Based Seismic Arrival-Time Picking Method. *Geophysical Journal International*, 216(1), 261–273. https://doi.org/10.1093/gji/ggy423
### INSTANCE

The tutorial uses PhaseNet weights pretrained on **INSTANCE**, an Italian seismic waveform dataset developed for machine-learning applications.

The complete INSTANCE dataset is not downloaded or retrained in this workflow. Only the pretrained weights distributed through SeisBench are used.

Reference:

> Michelini, A., Cianetti, S., Gaviano, S., Giunchi, C., Jozinović, D., and Lauciani, V. (2021). INSTANCE—The Italian Seismic Dataset for Machine Learning. *Earth System Science Data*, 13, 5509–5544. https://doi.org/10.5194/essd-13-5509-2021

### GaMMA

[GaMMA](https://github.com/AI4EPS/GaMMA) associates P- and S-wave picks recorded at multiple stations with individual seismic events.

The association uses:

- pick arrival times;
- phase types;
- pick probabilities;
- station coordinates;
- a one-dimensional velocity model;
- spatial search limits;
- DBSCAN pre-clustering;
- a Bayesian Gaussian Mixture Model.

GaMMA produces:

- preliminary origin times;
- preliminary hypocentral coordinates;
- pick-to-event associations;
- association probabilities.

Reference:

> Zhu, W., McBrearty, I. W., Mousavi, S. M., Ellsworth, W. L., and Beroza, G. C. (2022). Earthquake Phase Association Using a Bayesian Gaussian Mixture Model. *Journal of Geophysical Research: Solid Earth*, 127, e2021JB023249. https://doi.org/10.1029/2021JB023249
### PyProj

[PyProj](https://pyproj4.github.io/pyproj/) converts coordinates between:

- WGS84 geographical coordinates, EPSG:4326;
- UTM Zone 33N projected coordinates, EPSG:32633.

GaMMA operates on projected coordinates expressed in kilometres. Event locations are converted back to longitude and latitude for export and visualization.

### PyGMT

[PyGMT](https://www.pygmt.org/) is used to visualize:

- preliminary earthquake epicentres;
- event depths;
- station locations;
- topography;
- the geographical extent of the study area.

The current plotting workflow may require a local topographic grid.

### Additional packages

The notebooks also use:

- NumPy;
- pandas;
- Matplotlib;
- Seaborn;
- PyTorch;
- tqdm;
- scikit-learn dependencies used by GaMMA.

---
## Repository structure

The recommended repository structure is:

```text
EPOS_TUTORIAL_EGU2025/
├── README.md
├── environment.yml
│
├── 0_Introduction.ipynb
├── 1_download_mseed.ipynb
├── 2_apply_picking.ipynb
├── 3_build_the_catalog.ipynb
│
├── data_example/
│   └── Example input files
│
├── TEST_AMATRICE/
│   ├── waveforms/
│   │   └── MiniSEED waveform files
│   │
│   ├── inventory/
│   │   └── StationXML metadata
│   │
│   ├── stations.csv
│   │
│   └── output/
│       ├── pick_list/
│       │   ├── picks_<station>.csv
│       │   ├── all_picks_2016.dat
│       │   └── all_picks_2016_sort.dat
│       │
│       ├── catalog/
│       │   ├── seismic_catalog_2016.csv
│       │   ├── seismic_catalog_with_latlon_2016_instance.csv
│       │   ├── gamma_pick_2016_instance.csv
│       │   └── gamma_pick_grouped_2016_instance.csv
│       │
│       └── plot/
│           └── Diagnostic and catalog figures
│
└── download_log.txt
```
Waveform files may be excluded from the Git repository because of their size. They can be regenerated by running the download notebook.

---

## Notebook description

The notebooks must be executed in numerical order.

### `0_Introduction.ipynb`

Provides a concise introduction to:

- the Central Italy case study;
- the purpose of the tutorial;
- the processing workflow;
- the expected inputs and outputs;
- the distinction between a preliminary and a final seismic catalog.

This notebook should not duplicate the complete README and should not describe processing steps that are not implemented in the tutorial.
### `1_download_mseed.ipynb`

Retrieves seismic waveforms and station metadata.

The notebook:

1. defines the time interval and geographical region;
2. initializes the EPOS Seismology/ORFEUS–EIDA routing client;
3. optionally initializes an external EarthScope/IRIS fallback;
4. queries available stations and channels;
5. downloads continuous waveform data;
6. writes waveform files in MiniSEED format;
7. writes metadata in StationXML format;
8. records successful and failed requests in a log file;
9. plots example three-component waveforms.

Principal outputs:

```text
TEST_AMATRICE/waveforms/
TEST_AMATRICE/inventory/
download_log.txt
waveform_plot_<station>.pdf
```

### `2_apply_picking.ipynb`

Applies PhaseNet to three-component seismic waveforms through SeisBench.

The notebook:

1. loads PhaseNet pretrained with INSTANCE weights;
2. reads three-component waveform data;
3. removes duplicate traces;
4. calculates continuous P-, S- and noise-probability annotations;
5. extracts discrete picks using selected thresholds;
6. records station identifier, arrival time, probability and phase type;
7. saves one pick table for each station;
8. combines and chronologically sorts all picks.

Expected pick-table structure:

| Column | Description |
|---|---|
| `id` | Waveform or station identifier |
| `timestamp` | Predicted arrival time |
| `prob` | Maximum PhaseNet probability |
| `type` | Phase type: `p` or `s` |

Principal outputs:

```text
TEST_AMATRICE/output/pick_list/picks_<station>.csv
TEST_AMATRICE/output/pick_list/all_picks_2016.dat
TEST_AMATRICE/output/pick_list/all_picks_2016_sort.dat
TEST_AMATRICE/output/plot/random_trace_<station>.pdf
```

### `3_build_the_catalog.ipynb`

Associates PhaseNet picks with GaMMA and generates a preliminary catalog.

The notebook:


3. transforms coordinates from WGS84 to UTM Zone 33N;
4. defines spatial limits and association parameters;
5. defines a one-dimensional velocity model;
6. applies DBSCAN pre-clustering;
7. applies Bayesian Gaussian Mixture Model association;
8. estimates preliminary event origin times and hypocentral coordinates;
9. links individual picks to associated events;
10. converts projected event coordinates back to longitude and latitude;
11. saves catalog and association tables;
12. plots stations and preliminary seismicity.

Principal outputs:

```text
TEST_AMATRICE/output/catalog/seismic_catalog_2016.csv
TEST_AMATRICE/output/catalog/seismic_catalog_with_latlon_2016_instance.csv
TEST_AMATRICE/output/catalog/gamma_pick_2016_instance.csv
TEST_AMATRICE/output/catalog/gamma_pick_grouped_2016_instance.csv
TEST_AMATRICE/output/catalog/catalog_2016_instance_pygmt.pdf
```

---

## Installation

### Requirements

The workflow requires:

- Git;
- Anaconda or Miniconda;
- Python 3.9 or a compatible version;
- JupyterLab;
- internet access for waveform retrieval;
- internet access for the first download of pretrained model weights.

### Clone the repository

```bash
git clone https://github.com/RossellaFonzetti/EPOS_TUTORIAL_EGU2025.git
cd EPOS_TUTORIAL_EGU2025
```

### Create the environment

```bash
conda env create -f environment.yml
conda activate epos_tutorial
```

If the environment file retains its original name, use:

```bash
conda env create -f epos_tutorial.yml
conda activate epos_tutorial
### Start JupyterLab

```bash
jupyter-lab
```

### Optional manual installation

If GaMMA is not included in the Conda environment:

```bash
pip install git+https://github.com/AI4EPS/GaMMA.git
```

Principal packages can be installed manually with:

```bash
pip install obspy seisbench numpy pandas matplotlib seaborn pyproj tqdm
pip install git+https://github.com/AI4EPS/GaMMA.git
```

PyGMT may require a separate GMT installation. Follow the official PyGMT installation instructions for the operating system being used.

---

## Configuration

Before running the notebooks, verify:

- working directory;
- input and output paths;
- start and end times;
- geographical bounding box;
- network and channel selections;
- PhaseNet pretrained-model name;
- P- and S-pick thresholds;
- GaMMA association parameters;
- minimum number of P and S picks;
- velocity model;
- projected coordinate reference system;
- path to the topographic grid, if used.

### Portable paths

Avoid machine-specific absolute paths such as:

```python
base_dir = "/Users/username/path/to/EPOS_TUTORIAL_EGU2025"
```

Prefer:

```python
from pathlib import Path

base_dir = Path.cwd()
case_dir = base_dir / "TEST_AMATRICE"
```

This makes the notebooks easier to run on different computers.
---

## How to run the tutorial

Run the notebooks in this order:

```text
0_Introduction.ipynb
1_download_mseed.ipynb
2_apply_picking.ipynb
3_build_the_catalog.ipynb
```

The introductory notebook is optional for execution but recommended for understanding the case study and workflow.

The three operational notebooks are sequential:

1. the download notebook creates waveform and metadata files;
2. the picking notebook reads the downloaded waveforms;
3. the catalog notebook reads the generated pick tables.

Do not run the catalog-building notebook before the pick files have been generated.

---
## Output files

### Waveforms

Expected format:

```text
waveforms/
└── 2016/
    └── IV/
        └── CAMP/
            ├── HHZ.D/
            ├── HHN.D/
            └── HHE.D/
```

### Station table

The station table used by GaMMA must contain at least:

```text
network,station,latitude,longitude,elevation
```

Example:

```csv
network,station,latitude,longitude,elevation
IV,CAMP,42.5431,13.3472,850
```

Coordinates are expressed in decimal degrees and elevation in metres.

### Pick table

The combined pick table must contain:

```text
id,timestamp,prob,type
```

Example:

```csv
id,timestamp,prob,type
IV.CAMP.,2016-10-30 06:41:12.340000,0.94,p
IV.CAMP.,2016-10-30 06:41:18.760000,0.87,s
```

### Preliminary catalog
The GaMMA catalog includes estimated quantities such as:

- event index;
- origin time;
- projected coordinates;
- longitude and latitude;
- depth;
- number of associated picks;
- association-related metrics.

Column names may depend on the installed GaMMA version and the final export code.

---
## Limitations

The repository is intended for educational use.

Current limitations include:

- only a limited time interval is processed;
- data availability depends on selected networks and channels;
- missing components may prevent PhaseNet processing;
- waveform gaps may affect the output;
- picking results depend on pretrained weights and thresholds;
- association results depend on station geometry;
- association results depend on the velocity model;
- GaMMA locations are preliminary;
- magnitudes are not calculated;
- event relocation is not performed;
- no complete uncertainty analysis is implemented;
- no systematic comparison with an authoritative catalog is included;
- plotting may require a local topographic grid;
- results may vary with software and model versions.

For scientific use, the catalog should undergo:

- waveform quality control;
- phase-pick quality control;
- sensitivity tests on picking thresholds;
- verification of station metadata;
- evaluation of the velocity model;
- removal of false detections;
- refined earthquake location or relocation;
- magnitude calculation;
- comparison with an authoritative catalog;
- uncertainty assessment.

---
## References

### Data infrastructure and software

- [EPOS](https://www.epos-eu.org/)
- [EPOS Seismology](https://www.epos-eu.org/tcs/seismology)
- [ORFEUS–EIDA](https://www.orfeus-eu.org/data/eida/)
- [ObsPy](https://docs.obspy.org/)
- [SeisBench](https://github.com/seisbench/seisbench)
- [PhaseNet](https://github.com/AI4EPS/PhaseNet)
- [INSTANCE](https://github.com/ingv/instance)
- [GaMMA](https://github.com/AI4EPS/GaMMA)
- [PyProj](https://pyproj4.github.io/pyproj/)
- [PyGMT](https://www.pygmt.org/)

### Scientific references

Beyreuther, M., Barsch, R., Krischer, L., Megies, T., Behr, Y., and Wassermann, J. (2010). ObsPy: A Python Toolbox for Seismology. *Seismological Research Letters*, 81(3), 530–533. https://doi.org/10.1785/gssrl.81.3.530

Michelini, A., Cianetti, S., Gaviano, S., Giunchi, C., Jozinović, D., and Lauciani, V. (2021). INSTANCE—The Italian Seismic Dataset for Machine Learning. *Earth System Science Data*, 13, 5509–5544. https://doi.org/10.5194/essd-13-5509-2021

Woollam, J., Münchmeyer, J., Tilmann, F., Rietbrock, A., Lange, D., Bornstein, T., et al. (2022). SeisBench—A Toolbox for Machine Learning in Seismology. *Seismological Research Letters*, 93(3), 1695–1709. https://doi.org/10.1785/0220210324

Zhu, W., and Beroza, G. C. (2019). PhaseNet: A Deep-Neural-Network-Based Seismic Arrival-Time Picking Method. *Geophysical Journal International*, 216(1), 261–273. https://doi.org/10.1093/gji/ggy423

Zhu, W., McBrearty, I. W., Mousavi, S. M., Ellsworth, W. L., and Beroza, G. C. (2022). Earthquake Phase Association Using a Bayesian Gaussian Mixture Model. *Journal of Geophysical Research: Solid Earth*, 127, e2021JB023249. https://doi.org/10.1029/2021JB023249

---
---

## Citation

When using this repository, cite the repository release together with the software and methods used by the workflow.

Suggested repository citation:

> Fonzetti, R., and Bailo, D. (2025). *Deep-Learning Seismic Catalog Workflow for the 2016 Central Italy Sequence*. Tutorial presented at the EGU General Assembly 2025. Repository release: [add DOI or Zenodo record].

---

## Contributors

- **Rossella Fonzetti** — EPOS, Istituto Nazionale di Geofisica e Vulcanologia
  rossella.fonzetti@ingv.it

- **Daniele Bailo** — EPOS, Istituto Nazionale di Geofisica e Vulcanologia
  daniele.bailo@ingv.it

Questions, bug reports and suggestions can be submitted through the repository [issue tracker](https://github.com/RossellaFonzetti/EPOS_TUTORIAL_EGU2025/issues).

---

## License

© 2025 EPOS — European Plate Observing System.

This work is licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/), unless otherwise stated.

Individual software packages, data services, waveform data, station metadata and external images remain subject to their respective licenses and terms of use.


