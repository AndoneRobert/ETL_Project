# Formula 1 Session Data Pipeline

A local ETL workflow for cleaning, joining and summarising Formula 1 session data for the 2026 season. The pipeline is implemented in Pentaho Data Integration (PDI/Kettle) and works with CSV datasets stored in the project folder.

## Features

- Cleans and deduplicates circuit, meeting, session, driver, session-result, lap, stint and pit-stop records.
- Separates records that fail validation into rejection outputs where configured.
- Enriches session results, laps and stints with driver and team information.
- Adds session metadata to pit-stop records.
- Produces a driver/session summary with timed-lap count, average lap duration and fastest lap duration.
- Uses local data files at runtime; no live API connection is required to execute the pipeline.

## Requirements

- Pentaho Data Integration / Spoon 10.2, or a compatible version.
- A Java runtime supported by the installed Pentaho version.
- The supplied project data files in their expected directory structure.

The project has been tested on Linux.

## Run the pipeline

1. Extract the project archive and keep the folder structure intact.
2. Start Pentaho Spoon.
3. Open `dataintegration/00_run_etl.kjb`.
4. Run the Job using the local run configuration.
5. Review the Job log for failed entries or transformation errors.
6. Open `data/output/` to inspect the generated datasets and summaries.

Run the master Job rather than individual transformations when you want the complete pipeline to execute in sequence.

## Input data

Place the source CSV datasets in `data/input/` and keep their filenames and column headers consistent with the supplied files.

| File | Contents |
| --- | --- |
| `circuits.csv` | Circuit reference data |
| `meetings_2026.csv` | Grand Prix / meeting information |
| `sessions_2026.csv` | Session identifiers, names, types and dates |
| `drivers_2026.csv` | Driver and team information by session |
| `session_results_2026.csv` | Session-result records |
| `laps_2026.csv` | Lap and sector timing data |
| `stints_2026.csv` | Tyre stint records |
| `pit_stops_2026.csv` | Pit-stop measurements |

The OpenF1 API documentation is available at <https://openf1.org/docs/>.

## Processing stages

The master Job coordinates these transformations:

| Transformation | Purpose |
| --- | --- |
| `01_clean_circuits.ktr` | Validate, normalise and deduplicate circuits |
| `02_clean_sessions.ktr` | Validate and deduplicate sessions |
| `03_clean_drivers.ktr` | Normalise and deduplicate driver/session records |
| `04_clean_session_results.ktr` | Validate and deduplicate session results |
| `05_clean_laps.ktr` | Validate and deduplicate lap records |
| `06_clean_stints.ktr` | Validate and deduplicate stint records |
| `07_clean_pit_stops.ktr` | Validate and deduplicate pit-stop records |
| `08_clean_meetings.ktr` | Validate and deduplicate meetings |
| `09_combine_sessions_drivers.ktr` | Join driver/team data to session results |
| `10_combine_laps_drivers.ktr` | Join driver/team data to laps |
| `11_combine_stints_drivers.ktr` | Join driver/team data to stints |
| `12_combine_pit_stops_sessions.ktr` | Add session information to pit stops |
| `13_session_summary.ktr` | Aggregate lap metrics by session, driver and team |

## Output data

The pipeline writes its results to `data/output/`. Outputs include cleaned datasets (for example, `drivers_clean.csv`, `laps_clean.csv` and `pit_stops_clean.csv`), enriched datasets (such as `laps_with_drivers.csv` and `pit_stops_with_sessions.csv`), rejection files, and the session summary (`session_summary.txt`). Output files are regenerated when the corresponding transformations run.

For the session summary, laps without a usable duration are separated into `untimed_laps_rejected.csv`; valid timed laps contribute to the lap count, average duration and fastest duration.

## Troubleshooting

- **Input file not found:** confirm that the archive was extracted completely and that the CSV filenames match those listed above.
- **Field or schema errors:** check that the input CSV headers and column types match the supplied datasets.
- **A transformation fails:** inspect its log in Spoon and resolve the first reported error before rerunning the master Job.
- **Unexpected output counts:** check the Job log and verify that the input files belong to the same season and dataset snapshot.

## Demonstration

Video demonstration: <https://youtu.be/wczcOYMRy9k>

## Project information

- Author: Robert Andone
- Unit: Information Systems Integration, 2026/27
- Report: `doc/51242_doc.pdf`
