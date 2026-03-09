# Osa Conservation Camera Trap Database

This repository contains the data management system and automated web dashboard for Osa Conservation's camera trap network. Data are stored across four linked Google Sheets and visualised via a GitHub Pages site that updates daily.

---

## Database Structure

The database consists of four linked Google Sheets. Each sheet serves a distinct purpose and sheets are linked by shared key fields (`project_name`, `camera_location`, `camera_code`).

```
01_Project_details
       │
       │ project_name
       ▼
04_Deployment_data ──── camera_location ────▶ 03_Camera_locations
       │
       │ camera_code
       ▼
02_Camera_Inventory
```

---

### 01_Project_details

Stores one row per research project. This is the top-level table — all deployments must reference a project defined here.

| Column | Description |
|---|---|
| `project_name` | **Primary key.** Unique short code for the project (e.g. `long_term_monitoring`) |
| `project_folder` | Human-readable project name |
| `project_start` | Project start date |
| `project_end` | Project end date (blank if ongoing) |
| `project_target` | Target species or group (e.g. `multiple`) |
| `project_species_individual` | Whether individual identification is required |
| `project_bait_use` | Whether bait was used (`yes`/`no`) |
| `project_admin_email` | Contact email for project lead |
| `project_admin_name` | Name of project lead |
| `Comments` | Free-text notes |

---

### 02_Camera_Inventory

Stores one row per physical camera unit. Tracks hardware status, location, and purchase details.

| Column | Description |
|---|---|
| `camera_code` | **Primary key.** Unique camera identifier (e.g. `M500`) |
| `make` | Camera manufacturer (e.g. `Browning`) |
| `model` | Camera model |
| `serial_number` | Manufacturer serial number |
| `invoice_link` | Google Drive link to purchase invoice |
| `purchase_date` | Date of purchase |
| `purchased_by` | Funding source or purchaser |
| `status` | Current status (e.g. `functioning`, `broken`, `lost`) |
| `storage_location` | Where the camera is currently located (e.g. `field`, `office`) |
| `last_checked` | Date of last physical inspection |
| `checked_by` | Person who last checked the camera |
| `Comments` | Free-text notes |

---

### 03_Camera_locations

Stores one row per named camera trap station. Coordinates are fixed to the station, not to any individual deployment.

| Column | Description |
|---|---|
| `camera_location` | **Primary key.** Unique station code (e.g. `OCCT66`) |
| `longitude` | Decimal degrees longitude (WGS84) |
| `latitude` | Decimal degrees latitude (WGS84) |

---

### 04_Deployment_data

The central operational table. Stores one row per deployment event — each time a camera is placed at a location for a project. This table links all other sheets together.

| Column | Description |
|---|---|
| `project_name` | **Foreign key** → `01_Project_details.project_name` |
| `deployment_id` | Auto-generated unique ID (`camera_location_camera_code_startdate`) |
| `camera_location` | **Foreign key** → `03_Camera_locations.camera_location` |
| `camera_code` | **Foreign key** → `02_Camera_Inventory.camera_code` |
| `start_date` | Date camera was deployed |
| `start_time` | Time camera was deployed |
| `mode` | Capture mode (e.g. `photo`, `video`) |
| `end_date` | Date camera was retrieved |
| `end_time` | Time camera was retrieved |
| `Estado` | Camera status at retrieval (e.g. `Camera Functioning`, `Unknown Failure`) |
| `Batterias_%` | Battery level at retrieval |
| `end_comments` | Notes recorded at retrieval |
| `height_cm` | Camera mounting height in centimetres |
| `angle` | Camera angle |
| `quiet_period` | Quiet period setting (seconds) |
| `comentarios` | General field notes |
| `hard_drive_location` | Physical location of raw data |
| `backup_location` | Backup storage location |
| `processing_platform` | Platform used for image processing (e.g. `wildlife_insights`) |
| `uploaded` | Whether images have been uploaded (`Yes`/`No`) |
| `processed` | Whether images have been processed (`Yes`/`No`) |
| `other_comments` | Additional free-text notes |

---

## Data Entry Rules

- **No duplicate camera locations** — each `camera_location` code must appear only once in `03_Camera_locations`. Data validation is enforced in the sheet.
- **Dropdown validation** — `project_name`, `camera_location`, and `camera_code` fields in `04_Deployment_data` use dropdown validation linked via `IMPORTRANGE` to the relevant sheets, ensuring referential integrity.
- **Do not edit the `_Lookups` tab** — the hidden `_LookupsDONT TOUCH` tabs in sheets 02 and 04 are used to drive dropdown validation and should not be modified manually.

---

## Dashboard

The data are visualised at:

**[https://osa-conservation.github.io/Camera_trap_inventory_and_deployments/](https://osa-conservation.github.io/Camera_trap_inventory_and_deployments/)**

The dashboard is built with R Markdown and Leaflet, and updates automatically every day at 06:00 UTC via GitHub Actions. It includes:

- An interactive map of all camera locations, colour-coded by project, with toggleable project layers
- Searchable tables of projects, cameras, and deployments

---

## Repository Structure

```
.
├── index.Rmd                        # R Markdown source for the dashboard
├── .github/
│   └── workflows/
│       └── update-site.yaml         # GitHub Actions workflow (daily build)
├── docs/                            # Ignored locally, served via gh-pages branch
└── README.md
```

---

## Authentication

The dashboard fetches data from private Google Sheets using a Google Service Account. The service account credentials are stored as an organisation-level GitHub Secret and injected at build time. No credentials are stored in the repository.

To grant a new service account access, share each of the four Google Sheets with the service account email as a Viewer.
