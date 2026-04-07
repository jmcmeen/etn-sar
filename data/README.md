# Data Dictionary

## `herpetofauna.csv`

Species counts for four nested geographic areas in eastern Tennessee, compiled from published survey data.

| Column | Type | Description |
|---|---|---|
| `Locality` | string | Geographic area name |
| `Area` | float | Land area in km² |
| `TotalHerpetofauna` | int | Total amphibian + reptile species count |
| `Amphibians` | int | Amphibian species count (frogs + salamanders) |
| `Reptiles` | int | Reptile species count (lizards + snakes + turtles) |

### Spatial hierarchy

The four localities form a nested series: Steele Creek Park (9.3 km²) is within Sullivan County (1,114 km²), which is within the 5-county northeastern Tennessee region (4,137 km²), which is within eastern Tennessee (37,438 km²). "Northeastern Tennessee" refers to the counties of Sullivan, Washington, Carter, Johnson, and Unicoi.

### Provenance

Species counts are from the herpetofaunal inventories compiled in:

> Jessee, L.D., Stout, J.B. & McMeen, J.N. (2022). Herpetofauna of Steele Creek Park (Sullivan County, TN), with Comments on Species-Area Relationships of Amphibians and Reptiles in Eastern Tennessee. *Southeastern Naturalist*, 21(1), 63-73.
