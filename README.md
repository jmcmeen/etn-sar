# Species-Area Relationships of Amphibians and Reptiles in Eastern Tennessee

Artifacts and updated analyses from:

> **Jessee, L.D., Stout, J.B., & McMeen, J.N. (2022).** Herpetofauna of Steele Creek Park (Sullivan County, TN), with Comments on Species-Area Relationships of Amphibians and Reptiles in Eastern Tennessee. *Southeastern Naturalist*, 21(1), 63-73.

## About the Paper

Steele Creek Park is a 9.259-km² municipal park in Sullivan County, northeastern Tennessee. Surrounded by developed areas, the park's steep, forested, shale knobs form an ecological "island in the city" well suited to biogeographical research.

The study compiled a herpetofaunal inventory for the park (10 frogs, 11 salamanders, 2 lizards, 11 snakes, 7 turtles) and established species-area relationships (SARs) for amphibians and reptiles across four nested sampling areas ranging from Steele Creek Park to all of eastern Tennessee. The amphibian model (Adj. R² = 0.98) proved highly predictive; the reptile model (Adj. R² = 0.33) was weaker, suggesting factors beyond area drive reptile diversity in the region.

## Repository Contents

```text
data/
  herpetofauna.csv           Nested area-richness data (4 sites)
publication/
  SN_2022_Jessee_et_al.pdf   Published paper
  Jessee_et_al_2022.md       Paper in markdown format
  figures/                   Figures from the publication
analysis.ipynb               Reproduction and extension of the SAR analysis
analysis_updates.ipynb       Additional analyses beyond the original paper
```

## Data

The core dataset (`data/herpetofauna.csv`) contains species counts for four nested geographic areas:

| Locality | Area (km²) | Amphibians | Reptiles | Total |
| --- | ---: | ---: | ---: | ---: |
| Steele Creek Park | 9.3 | 20 | 21 | 41 |
| Sullivan County | 1,114 | 38 | 21 | 59 |
| Northeastern Tennessee | 4,137 | 44 | 24 | 68 |
| Eastern Tennessee | 37,438 | 69 | 44 | 113 |

## Analysis

The main notebook (`analysis.ipynb`) reproduces and extends the paper's SAR analysis:

1. **Log-log regression** of species richness on area for total herpetofauna, amphibians, and reptiles
2. **Power model derivation** (S = CA^z) from the regression parameters
3. **Model testing** against independent sites (Great Smoky Mountains NP, Cumberland Gap NP)

The amphibian model predicted 42.7 species for Great Smoky Mountains NP compared to 43 reported, demonstrating strong predictive accuracy.

## Setup

```bash
pip install -r requirements.txt
jupyter notebook analysis.ipynb
```
