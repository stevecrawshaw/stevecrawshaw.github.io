---
title: "Defining Growth Zone Boundaries with LSOA Analysis"
date: 2026-05-28
categories: [R]
tags:
- r
- spatial
- tmap
- sf
- quarto
- growth-zones
- weca
---

**TLDR; WECA's Growth Strategy identifies five growth zones but gives only illustrative maps. This report works through a practical method for translating those sketched boundaries into precise LSOA-based geographies suitable for economic and demographic analysis.**

## The problem

The [West of England Growth Strategy](https://www.westofengland-ca.gov.uk/wp-content/uploads/2025/09/WE4915-Growth-Strategy-exec-summary_aw_web_v1.pdf) published in March 2026 identifies five growth zones — the Western Innovation Arc, Central Bristol and Bath, Severn Estuary, Somer Valley, and North Somerset Growth Gateway — as the engine rooms of regional investment and development. The strategy includes illustrative maps, but these are not precise enough to link to census data, deprivation indices, employment statistics, or any of the standard analytical datasets that are organised around official statistical geographies.

To do any meaningful economic analysis — population, housing, jobs, deprivation — you need boundaries that snap to real data geographies.

## Why LSOAs?

Lower Super Output Areas are the standard workhorse geography for small-area analysis in England. They have between 1,000 and 3,000 residents, were designed to be socially homogeneous, and are the smallest geography for which a wide range of data is consistently published. Output Areas are smaller but have data suppression problems; MSOAs and wards are too coarse to capture the edge of a growth zone accurately.

The 2021 Census LSOA boundaries are used throughout.

## Methodology

The process runs in two stages:

**1. Digitise the illustrative boundaries.** The growth zone outlines from the Growth Strategy were traced by eye in GIS to create polygon layers. These are rough approximations — the point is to have something to intersect against, not to claim the digitised line is definitive.

**2. Intersect with LSOA population-weighted centroids.** A simple any-intersection approach pulls in too many LSOAs that only clip a corner of the growth zone. Using the population-weighted centroid (PWC) of each LSOA instead — including an LSOA only if its centroid falls inside the digitised boundary — produces a much tighter, more honest result. It effectively answers "does the majority of this LSOA's population sit inside the growth zone?"

One exception applies: the Bristol to Bath Corridor is a linear feature along the A4/railway, and the centroid method excludes key LSOAs within it. The illustrative boundary was minimally redigitised along the corridor to capture those centroids.

## Boundary issues and proposals

The LSOA analysis throws up a dozen specific boundary questions — LSOAs that straddle two zones, low-density areas where the centroid method systematically under-represents economic activity (Severn Estuary port and industrial land), and ambiguous edges where the illustrative boundary is genuinely unclear. Each issue is documented with a proposal or open question and an interactive map.

The [full interactive report](/demos/gz-boundary-evolution.html) covers all of these in detail, including:

- Midsomer Norton (accept PWC boundary)
- North Somerset Growth Gateway split into Mendip and Weston Urban sub-zones
- Brislington allocated site inclusion in Central Bristol
- Bristol to Bath Corridor extent and infill LSOAs at Pixash and Hick's Gate
- Severn Estuary extension to include Royal Portbury Dock
- Oldbury opportunity area options (full LSOA vs constrained boundary)
- Bath City Centre extent
- Western Innovation Arc eastern end and UWE Frenchay Campus
- WIA / Severn Estuary zone delineation at the Brabazon / Aztec West LSOA

## The report

The analysis is written in Quarto with R, using `sf` for spatial operations and `tmap` in interactive mode for the maps. All maps are zoomable and show both the proposed LSOA boundaries and the original illustrative digitisation for comparison.

**[Open the interactive report →](/demos/gz-boundary-evolution.html)**

*Note: the report is large (~65MB) due to embedded interactive maps. Give it a moment to load.*
