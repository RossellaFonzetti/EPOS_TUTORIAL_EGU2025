# EPOS_TUTORIAL_EGU2025

Welcome to the **EPOS Tutorial** repository for the *EGU General Assembly 2025*!

This repository contains the materials, notebooks, and instructions for the *EPOS Tutorial* held during EGU 2025. The aim is to guide users through the integration and use of EPOS research infrastructures and services with a practical, hands-on approach.

## Overview

In this tutorial, participants will learn how to:

•⁠  ⁠Access and use EPOS data and services
•⁠  ⁠Reproduce simple geophysical workflows
•⁠  ⁠Explore cross-disciplinary integration of data
•⁠  ⁠Run Jupyter Notebooks with real data

All examples are tailored to promote **FAIR data** practices and reproducible science.

## Requirements

To run the tutorial materials locally, you need:

•⁠  ⁠[Anaconda or Miniconda](https://docs.conda.io/en/latest/)

•⁠  ⁠Git

•⁠  ⁠Python 3.9+

## Setup Instructions
```python
⁠bash
git clone https://github.com/RossellaFonzetti/EPOS_TUTORIAL_EGU2025.git
cd EPOS_TUTORIAL_EGU2025
conda env create -f epos_tutorial.yml
conda activate epos_tutorial
jupyter-lab
```

## Contributors

•⁠  ⁠*Rossella Fonzetti* – EPOS, INGV rossella.fonzetti@ingv.it

•⁠  ⁠*Daniele Bailo* – EPOS, INGV daniele.bailo@invg.it

For any questions or feedback, feel free to open an [issue](https://github.com/RossellaFonzetti/EPOS_TUTORIAL_EGU2025/issues) or contact us directly.

---

©️ 2025 | EPOS – European Plate Observing System  
This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

## INTRODUCTION TO THE CASE STUDY

This tutorial explains step by step the creation of seismic catalog using deep learning methods.


The case study is the 2016-2017 Amatrice-Visso-Norcia seismic sequence (Central Italy).

The 2016 Central Italy seismic sequence is one of the most significant recent seismic crises in Europe. It began on **August 24, 2016**, with a magnitude Mw 6.0 earthquake near Amatrice, followed by several major events in the following months (e.g., Mw 5.9 on Oct 26, Mw 6.5 on Oct 30).

![Seismicity Map - Central Italy 2016](https://upload.wikimedia.org/wikipedia/commons/a/a4/30-10-2016_central_italy_ShakeMap.jpg)

*Figure: Shake map of the 2016/10/30 Mw 6.5 Central Italy sequence (source: INGV via Wikipedia*)


List of the major earthquakes of the 2016–2017 Central Italy Sequence

| Date             | Local Time (CET/CEST) | Magnitude (Mw) | Depth (km) | Epicenter Municipality          | Latitude | Longitude |
|------------------|------------------------|----------------|------------|--------------------------------|----------|-----------|
| 24 Aug 2016      | 03:36:32               | 6.0            | 8          | Accumoli                       | 42.70 N  | 13.23 E   |
| 24 Aug 2016      | 04:33:28               | 5.3            | 8          | Norcia                         | 42.79 N  | 13.15 E   |
| 26 Oct 2016      | 19:10:36               | 5.4            | 9          | Castelsantangelo sul Nera      | 42.88 N  | 13.13 E   |
| 26 Oct 2016      | 21:18:05               | 5.9            | 8          | Ussita                         | 42.91 N  | 13.13 E   |
| 30 Oct 2016      | 07:40:17               | 6.5            | 9          | Norcia                         | 42.83 N  | 13.11 E   |
| 18 Jan 2017      | 10:25:40               | 5.1            | 10         | Montereale                    | 42.55 N  | 13.28 E   |
| 18 Jan 2017      | 11:14:09               | 5.5            | 10         | Capitignano                   | 42.53 N  | 13.28 E   |
| 18 Jan 2017      | 11:25:23               | 5.4            | 9          | Pizzoli                       | 42.50 N  | 13.28 E   |
| 18 Jan 2017      | 14:33:36               | 5.0            | 10         | Cagnano Amiterno              | 42.47 N  | 13.28 E   |

These events significantly affected Lazio, Marche, Umbria, and Abruzzo, causing extensive damage and casualties. The Mw 6.5 earthquake on October 30, 2016, was the strongest in Italy since 1980 (and it will be the case study of this tutorial).
