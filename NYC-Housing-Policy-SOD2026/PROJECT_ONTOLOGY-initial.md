# NYC Housing Policy Analysis — Project Ontology

*This document was produced using Claude, an AI assistant by Anthropic. Content should be reviewed for accuracy.*

---

## 1. Dataset Inventory

| # | Dataset | File(s) | Rows | Columns | Time Span | Grain |
|---|---------|---------|------|---------|-----------|-------|
| 1 | **NYC 311 Service Requests** | `NYC311-2010-to-20260301.csv` (25 GB) | ~42.96M | 44 | 2010 – Mar 2026 | Individual complaint |
| 2 | **Housing Database (DCP)** | `HousingDB_post2010.csv` (46 MB) | ~81.7K | 63 | Post-2010 jobs | DOB job application |
| 3 | **PLUTO** | `Primary_Land_Use_Tax_Lot_Output_(PLUTO)_20260206.csv` (411 MB) | ~858K | 101 | Snapshot (Feb 2026) | Tax lot (BBL) |
| 4 | **Furman Center SHD — BBL Analysis** | `FC_SHD_bbl_analysis_2025-05-13.csv` (5.6 MB) | ~13.9K | 126 | As of May 2025 | Subsidized tax lot |
| 5 | **Furman Center SHD — Subsidy Analysis** | `FC_SHD_subsidy_analysis_2025-05-13.csv` (6.1 MB) | ~14.1K | 124 | As of May 2025 | Individual subsidy record |
| 6 | **StreetEasy Master Report** | 44 zip archives in `StreetEasy_Master_Report/` | ~198 rows each | ~195 month-columns | Jan 2010 – Feb 2026 | Area × month (wide format) |

**Data Dictionaries (xlsx):** `Housing_Database_Data_Dictionary.xlsx`, `FC_SHD_bbl_analysis_data_dictionary_2025-05-13.xlsx`, `FC_SHD_subsidy_analysis_data_dictionary_2025-05-13.xlsx`

---

## 2. Entity Definitions

### 2.1 Tax Lot (BBL)
The fundamental spatial unit across most datasets. A **Borough-Block-Lot** (BBL) is a 10-digit identifier (1-digit borough + 5-digit block + 4-digit lot) assigned by the NYC Department of Finance. Present in: PLUTO (`BBL`), HousingDB (`BBL`), Furman Center BBL (`bbl`), 311 (`BBL`).

### 2.2 Building (BIN)
A **Building Identification Number** uniquely identifies a structure. Present in: HousingDB (`BIN`). A single BBL may contain multiple BINs.

### 2.3 DOB Job Application
A permit application filed with the Department of Buildings for new construction, alteration, or demolition. The primary entity in HousingDB, keyed by `Job_Number`.

### 2.4 311 Service Request
An individual complaint or service request filed through the NYC 311 system, keyed by `Unique Key`. Contains complaint type, location (address, BBL, lat/lon), timestamps, and resolution info.

### 2.5 Subsidized Housing Property
A residential property receiving one or more government subsidies. The Furman Center BBL file is at the property level; the Subsidy file has one row per subsidy-property pair (a single BBL may have multiple active subsidies).

### 2.6 StreetEasy Market Area
A named geographic area (neighborhood, submarket, borough, or citywide) for which StreetEasy reports monthly market metrics. Areas are identified by `areaName`, `Borough`, and `areaType`.

---

## 3. Key Attributes by Dataset

### 3.1 NYC 311 Service Requests (44 columns)
- **Identity:** Unique Key
- **Temporal:** Created Date, Closed Date, Due Date, Resolution Action Updated Date
- **Classification:** Problem (Complaint Type), Problem Detail (Descriptor), Additional Details, Agency, Agency Name, Location Type
- **Geography:** Incident Zip, Incident Address, Street Name, Cross Streets, City, Borough, Community Board, Council District, Police Precinct, BBL, Latitude, Longitude, X/Y Coordinates
- **Status:** Status, Resolution Description
- **Other:** Open Data Channel Type, Park Facility Name, Vehicle/Taxi/Bridge fields

**Sample agencies:** HPD, DOB, NYPD, DEP, DSNY, DOT, DOHMH, DPR, DCA, DHS, DFTA, TLC, EDC
**Sample complaint types (housing-relevant):** HEAT/HOT WATER, DOOR/WINDOW, ELECTRIC, ELEVATOR, FLOORING/STAIRS, GENERAL, Boilers, Building/Use, General Construction/Plumbing, Asbestos, Lead, Noise, Dirty Conditions

### 3.2 Housing Database — DCP (63 columns)
- **Identity:** Job_Number, BIN, BBL
- **Job classification:** Job_Type (New Building / Alteration / Demolition), Job_Status, ResidFlag, NonresFlag
- **Unit counts:** ClassAInit, ClassAProp, ClassANet, HotelInit, HotelProp, OtherBInit, OtherBProp, Units_CO
- **Temporal:** DateFiled, DatePermit, DateComplt, DateLstUpd, PermitYear, CompltYear
- **Building:** Occ_Init, Occ_Prop, Bldg_Class, FloorsInit, FloorsProp, Enlargemnt, Ownership, Job_Desc
- **Zoning:** ZoningDst1–3, SpeclDst1–2, Landmark
- **Geography:** Boro, AddressNum, AddressSt, Latitude, Longitude, CenTract20, NTA2020, NTAName20, CDTA2020, CDTAName20, CommntyDst, CouncilDst, PolicePcnt, SchSubDist, SchCommnty, FireCmpany, etc.
- **Flood risk:** PL_FIRM07, PL_PFIRM15

### 3.3 PLUTO (101 columns)
- **Identity:** BBL (borough + Tax block + Tax lot), borocode
- **Land use:** landuse, bldgclass, zonedist1–4, overlay1–2, spdist1–3, splitzone, ltdheight
- **Physical:** lotarea, bldgarea, comarea, resarea, officearea, retailarea, garagearea, strgearea, factryarea, otherarea, numbldgs, numfloors, unitsres, unitstotal, lotfront, lotdepth, bldgfront, bldgdepth
- **Financial:** assessland, assesstot, exempttot
- **Age/history:** yearbuilt, yearalter1, yearalter2
- **Ownership:** ownertype, ownername
- **Districts:** community board, council district, schooldist, firecomp, policeprct, healtharea, sanitboro, sanitsub, postcode
- **Landmarks/historic:** histdist, landmark
- **FAR:** builtfar, residfar, commfar, facilfar
- **Flood:** firm07_flag, pfirm15_flag
- **Geography:** address, latitude, longitude, xcoord, ycoord, census tract 2010, bct2020, bctcb2020, geom

### 3.4 Furman Center — BBL Analysis (126 columns)
- **Identity:** bbl, standard_address
- **Property attributes:** res_units, year_built, buildings, assessed_value, owner_name
- **Condition flags:** ser_violation (serious HPD violations), tax_delinquency
- **Financial performance:** net_inc_sqft, gross_exp_sqft
- **Geography:** boro_id/name, cd_id/name, sba_id/name, ccd_id/name, city_id/name, tract_10, latitude, longitude
- **Income targeting:** inc_tar_units_0bed through 4bed, inc_tar_info
- **Data source flags (0/1):** data_dof, data_hcrlihtc, data_hpd, data_hudcon, data_hudfin, data_hudlihtc, data_ml, data_nycha
- **Subsidy category flags (0/1):** sub_fin, sub_prog, sub_tax, sub_zone
- **Program-level detail (28 programs × 3 columns each = 84 columns):** prog_*, start_*, end_* for: 202/8, 221d, 223, 420c, 421a, 421a_aff, 8a, hpd_oth, hud_fin_oth, hud_proj_oth, inclus_hous, j51, lamp, lihtc4, lihtc9, lmsa, mfp, ml, nep, nrp, nycha_mix, nycha_ph, plp, prac_202, proj8, rad, tpt, year15

### 3.5 Furman Center — Subsidy Analysis (124 columns)
Same structure as BBL Analysis but keyed by `fc_subsidy_id` (one row per subsidy). Adds: agency_name, subsidy_name, sub_subsidy_name, project_name, start_date, end_date, tenure, preservation, program, reac_score, reac_date, tot_bbls, tot_buildings, tot_units

### 3.6 StreetEasy Master Report (44 files)

**Sales Market Metrics** (by property type: All, Condo, Coop, Sfr):
- medianAskingPrice — Median listing price ($)
- medianSalesPrice — Median recorded sale price ($)
- recordedSalesVolume — Number of recorded sales
- totalInventory — Active for-sale listings
- daysOnMarket — Median days on market
- priceCutShare — Share of listings with a price cut
- saleListRatio — Sale-to-list price ratio
- priceIndex — StreetEasy Price Index (All only)

**Rental Market Metrics** (by bedroom count: All, Studio, OneBd, TwoBd, ThreePlusBd):
- medianAskingRent — Median asking rent ($)
- rentalInventory — Active rental listings
- discountShare — Share of rentals with a discount
- rentalIndex — StreetEasy Rental Index (All only)

**Structure:** Wide-format time series. Columns: `areaName`, `Borough`, `areaType`, then one column per month (YYYY-MM). Index files (priceIndex, rentalIndex) have columns: `month`, `Brooklyn`, `Manhattan`, `NYC`, `Queens`.

**Geographic levels (areaType):** city, borough, submarket, neighborhood (~198 areas)
**Boroughs:** Manhattan, Brooklyn, Queens, Bronx, Staten Island

---

## 4. Join Keys & Linkage Map

```
                    ┌──────────────┐
                    │   BBL (10d)  │  ← Primary spatial join key
                    └──────┬───────┘
           ┌───────────────┼───────────────────┐
           │               │                   │
     ┌─────▼─────┐   ┌────▼─────┐    ┌────────▼────────┐
     │   PLUTO   │   │ HousingDB│    │ Furman Ctr (BBL) │
     │  858K lots │   │ 81K jobs │    │  13.9K subsidized│
     └─────┬─────┘   └────┬─────┘    └────────┬────────┘
           │               │                   │
           │          BIN ─┘         fc_subsidy_id
           │                                   │
           │                          ┌────────▼────────┐
           │                          │ Furman Ctr (Sub) │
           │                          │ 14.1K subsidies  │
           │                          └─────────────────┘
           │
     ┌─────▼──────────┐
     │   NYC 311       │
     │  43M complaints │  ← joins on BBL (28% populated),
     │                 │    or Incident Zip / Borough /
     └────────────────┘    Community Board / Council Dist
```

**StreetEasy** does not share BBL. Linkage to lot-level data requires geographic bridging:

| StreetEasy areaType | Bridge to lot-level data via |
|---------------------|------------------------------|
| borough | PLUTO `borocode`, 311 `Borough` |
| neighborhood | Manual or fuzzy mapping to NTA / community district names |
| submarket | Manual mapping (StreetEasy-proprietary groupings) |
| city | N/A (citywide) |

**Cross-dataset join keys summary:**

| Key | PLUTO | HousingDB | FC-BBL | FC-Sub | 311 | StreetEasy |
|-----|-------|-----------|--------|--------|-----|------------|
| BBL | ✅ PK | ✅ | ✅ PK | ✅ (ref_bbl) | ✅ (partial) | — |
| BIN | — | ✅ | — | — | — | — |
| Borough | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Community District | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Council District | ✅ | ✅ | ✅ | ✅ | ✅ | — |
| Census Tract | ✅ | ✅ | ✅ | ✅ | — | — |
| NTA | — | ✅ | — | — | — | ~ (neighborhood) |
| Zip Code | ✅ | — | — | — | ✅ | — |
| Lat/Lon | ✅ | ✅ | ✅ | — | ✅ | — |

---

## 5. Temporal Coverage

| Dataset | Start | End | Granularity |
|---------|-------|-----|-------------|
| NYC 311 | 2010 | Mar 2026 | Daily (Created/Closed dates) |
| HousingDB | Post-2010 | Current vintage | Event dates (Filed, Permit, Complete) |
| PLUTO | Snapshot | Feb 2026 | Point-in-time |
| Furman Center | Varies by subsidy | May 2025 vintage | Subsidy start/end dates |
| StreetEasy | Jan 2010 | Feb 2026 | Monthly |

---

## 6. Thematic Domains & Analytical Dimensions

### 6.1 Housing Supply Pipeline
**Primary source:** HousingDB
**Key measures:** ClassANet (net new units), Job_Type, Job_Status, CompltYear, PermitYear
**Supports:** Tracking new construction, alterations, demolitions; net unit change by geography and year

### 6.2 Land Use & Zoning Capacity
**Primary source:** PLUTO
**Key measures:** landuse, bldgclass, zonedist, builtfar vs residfar/commfar, lotarea, bldgarea, unitsres
**Supports:** Zoning analysis, underbuilt lot identification, land use classification

### 6.3 Subsidized & Affordable Housing
**Primary sources:** Furman Center BBL + Subsidy
**Key measures:** Subsidy programs (28 types), start/end dates, res_units, income targeting, REAC scores, tenure, preservation vs. new construction
**Supports:** Subsidy expiration risk analysis, portfolio composition, geographic concentration, physical condition (REAC)

### 6.4 Housing Conditions & Complaints
**Primary source:** NYC 311
**Key measures:** Complaint type (HEAT/HOT WATER, etc.), volume trends, resolution time, geographic clustering
**Supports:** Identifying problem buildings/areas, correlating complaints with subsidized housing, seasonal patterns

### 6.5 Market Dynamics
**Primary source:** StreetEasy
**Key measures:** Median prices/rents, inventory levels, days on market, price cuts, sale/list ratios, price/rental indices
**Supports:** Price trend analysis by neighborhood/borough, market tightness indicators, affordability benchmarking

### 6.6 Property Characteristics & Valuation
**Primary source:** PLUTO
**Key measures:** assessland, assesstot, exempttot, yearbuilt, ownername, ownertype
**Supports:** Tax base analysis, age of housing stock, ownership patterns

### 6.7 Flood Risk
**Primary sources:** PLUTO (firm07_flag, pfirm15_flag), HousingDB (PL_FIRM07, PL_PFIRM15)
**Supports:** Identifying housing in flood zones, climate risk overlay with subsidized housing

---

## 7. Geographic Hierarchy

```
NYC (city)
 └── Borough (5)
      └── Community District (~59)
           └── Neighborhood / NTA (~262)
                └── Census Tract (~2,168)
                     └── Census Block
                          └── Tax Lot (BBL, ~858K)
```

Additional administrative overlays: Council District, Police Precinct, School District, Fire Company, Sanitation District, Health Area, Zip Code.

StreetEasy uses its own geography: city → borough → submarket → neighborhood (~198 areas total).

---

## 8. Subsidy Program Taxonomy (Furman Center)

**By category:**
- **Financing (sub_fin):** Section 221d, Section 223, LAMP, Multi-Family Program, Other HUD Financing
- **City/State Programs (sub_prog):** Mitchell-Lama, Article 8A/HRP, NEP, NRP, PLP, TPT, Other HPD Programs, NYCHA (Public Housing & Mixed Finance), RAD
- **Tax Incentives (sub_tax):** 421-a, 421-a Affordable, 420-c, J-51
- **Zoning (sub_zone):** Inclusionary Housing
- **Federal project-based:** Section 202/8, Project-Based Section 8, PRAC/202, LMSA, Other HUD Project-Based
- **Tax credits:** LIHTC 4%, LIHTC 9%, LIHTC Year 15

**Data sources:** DOF, HCR-LIHTC, HPD, HUD Contracts, HUD Financing, HUD-LIHTC, Mitchell-Lama, NYCHA

---

## 9. Known Data Considerations

1. **311 BBL coverage:** The BBL field in 311 data is not universally populated — geographic joins will have partial coverage; spatial joins via lat/lon can supplement.
2. **StreetEasy geographic bridging:** StreetEasy neighborhoods don't map 1:1 to official NTAs or community districts. A crosswalk or fuzzy mapping will be needed for lot-level integration.
3. **StreetEasy wide format:** All StreetEasy CSVs use wide format (one column per month). They will need to be unpivoted/melted for time-series analysis.
4. **PLUTO is a snapshot:** PLUTO represents current conditions only. Historical changes must be inferred from HousingDB or prior PLUTO vintages.
5. **Furman Center subsidy dates:** Some subsidy end dates may be missing or represent regulatory expiration rather than actual exit from affordability.
6. **311 file size:** At 25 GB and ~43M rows, the 311 file will require chunked processing or sampling for exploratory analysis.
7. **HousingDB includes inactive jobs:** The `Job_Status` field (and the noted `Job_Inactv` flag) should be used to filter to active/completed projects for pipeline analysis.

*This document was produced using Claude, an AI assistant by Anthropic. Content should be reviewed for accuracy.*
