# HoloLens 2 gaze recordings of four activities at an assembly workstation

This repository contains the dataset of the paper:

> **Gaze-Based Worker Activity Recognition in Industrial Assembly Using Temporal and Expert
> Feature Fusion**
> Pre-print on arXiv. The link and full citation will be added here once the pre-print is online.

It contains eye-tracking (gaze) recordings of ten participants, each performing four
activities at a workstation: assembling, gluing, waiting, and walking. The data are free to
use under the CC BY 4.0 license (see [License and consent](#license-and-consent)).

---

## What the repository contains

| Item | Content |
|---|---|
| `P01/` to `P10/` | One folder per participant, each with four CSV files, one per activity: `Pxx_Assembly.csv`, `Pxx_Gluing.csv`, `Pxx_Waiting.csv`, `Pxx_Walking.csv`. |
| `recordings.csv` | An index with one row per recording file (see [The index file](#the-index-file-recordingscsv)). |
| `LICENSE` | The full text of the CC BY 4.0 license. |

In total there are 40 recordings with 91,211 gaze samples, about 48 minutes of data. Each file
is one continuous recording of one activity, between 57 and 107 seconds long.

---

## How the data were recorded

- **Participants.** Ten volunteers, identified only by the codes P01 to P10.
- **Device.** A Microsoft HoloLens 2 headset. Each participant put on the headset, and then
  its built-in eye calibration was run.
- **Software.** Gaze was logged with ARETT, the Augmented Reality Eye Tracking Toolkit [1],
  at the headset's nominal rate of 30 samples per second (30 Hz).
- **Place.** A workstation in the smart factory of the German Research Center for Artificial
  Intelligence (DFKI) in Kaiserslautern, Germany.
- **Activities.**
  - *Assembling*: joining 3D-printed parts (a hood and a cargo bed) onto a wheeled toy-truck
    chassis.
  - *Gluing*: joining a 3D-printed lid and cube with a hand-held glue gun.
  - *Waiting*: standing at the workstation without a specified object to look at.
  - *Walking*: walking around the workstation area without a destination or work goal.

---

## File format

Each recording is a comma-separated (CSV) file with one row per gaze sample and 61 columns.
The column names and their meaning follow Table 1 of the ARETT paper [1]. Missing values are
written as `NA`, and true/false values as `True` and `False`.

The columns most users need:

| Column | Meaning |
|---|---|
| `eyeDataTimestamp` | Time of the sample in milliseconds since the start of the recording |
| `eyeDataRelativeTimestamp` | Time of the sample from the headset's high-precision clock, in milliseconds since the start of the recording |
| `frameTimestamp` | Time of the display frame in which the sample was processed, in milliseconds on the same scale as `eyeDataTimestamp` |
| `isCalibrationValid` | `True` if the headset's eye calibration was valid |
| `gazeHasValue` | `True` if the sample contains a valid gaze origin and direction |
| `gazeOrigin_x`, `_y`, `_z` | Start point of the gaze ray, in the global (room-fixed) coordinate system of the scene. On HoloLens 2, one unit is one meter |
| `gazeDirection_x`, `_y`, `_z` | Direction of the gaze ray in the same coordinate system, as a vector of length 1 |

Because the coordinates are fixed to the room, not to the head, the gaze origin and direction
also change when the participant moves or turns their head.

The other columns describe where the gaze ray hit virtual objects, screens or areas of interest.
No such objects were used in these recordings, so these columns are almost always `NA` or
`False`.

---

## The index file `recordings.csv`

| Column | Meaning |
|---|---|
| `file` | Path of the recording file |
| `slot` | Participant folder that the file is in |
| `recorded_by` | Participant who was actually recorded (differs from `slot` in two files; see [Before you analyze](#before-you-analyze-the-data)) |
| `activity` | Assembly, Gluing, Waiting, or Walking |
| `samples` | Number of rows (gaze samples) in the file |
| `duration_s` | Length of the recording in seconds |
| `valid_gaze_pct` | Percentage of samples with `gazeHasValue` = `True` |
| `repeated_rows` | Number of rows that repeat the previous row exactly |
| `max_gap_s` | Longest time between two consecutive samples, in seconds |
| `paper_file` | Name of the same recording in the paper's dataset |
| `notes` | Remarks on the recording |

---

## Before you analyze the data

1. **Two files were recorded by a different participant.** `P04/P04_Waiting.csv` was recorded
   by P10, and `P05/P05_Walking.csv` by P09, because P04's own waiting recording and P05's own
   walking recording contain no samples. If you need to keep people apart, for example to test
   on participants not seen in training, group the files by the `recorded_by` column, not by
   folder.
2. **Repeated rows in two files.** In `P09/P09_Waiting.csv` and `P09/P09_Walking.csv`, many rows
   are exact repeats of the row before (3,747 and 1,637 rows). They come from a logging fault
   and are kept as recorded. Remove them before any analysis that depends on time. After
   removal, these two files have fewer samples per second than the others.
3. **Small timestamp irregularities.** In seven files, one sample has an `eyeDataTimestamp`
   1 ms earlier than the sample before it (listed in `notes`). `eyeDataRelativeTimestamp` never
   decreases. The time between samples is about 33 ms but not exactly constant, so use the
   timestamps, not the row number, as the time axis.
4. **Invalid samples.** Samples with `gazeHasValue` = `False` (for example during blinks or when
   tracking was lost) have no gaze origin or direction. The share of valid samples is in the
   `valid_gaze_pct` column of `recordings.csv`.

---

## How to use the data

Any tool that reads CSV files works. A minimal example in Python with the pandas library:

```python
import pandas as pd

df = pd.read_csv("P01/P01_Walking.csv")    # "NA" is read as missing; True/False as booleans
df = df.drop_duplicates()                  # removes the repeated rows (only P09 has them)
valid = df[df["gazeHasValue"]]             # keep samples with a valid gaze ray
time_s = valid["eyeDataTimestamp"] / 1000  # time axis in seconds
direction = valid[["gazeDirection_x", "gazeDirection_y", "gazeDirection_z"]]
```

To work with all 40 recordings, read `recordings.csv` and loop over its `file` column. The
`recorded_by` and `activity` columns give the participant and the activity label of each file.

---

## License and consent

The recordings and their documentation are licensed under the
[Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).
The full text is in [`LICENSE`](LICENSE). You may copy, share, and adapt the data for any
purpose, including commercial use, as long as you give appropriate credit, link to the
license, and state whether you made changes. When you use the data, please cite the paper above.

All ten participants gave informed consent to the use of their recordings in the paper and to
their public release under CC BY 4.0.

## Privacy

The dataset contains no information that identifies a participant:

- Participants are identified only by the codes P01 to P10. The key that links codes to
  people is kept privately by the authors and will not be published.
- The ARETT metadata files were not included, because they contain participant names and
  recording dates.
- All timestamps were shifted so that each recording starts at 0. The original dates and
  times of the recordings cannot be recovered.
- The data contain only gaze measurements. There are no images, video, audio, or personal
  details.

Please do not try to identify the participants.

The timestamp shift and the moving of a misplaced header line to the top of the two P09 files
are the only changes made to the recorded values. Everything else is as exported by ARETT.

---

## Reference

[1] S. Kapp, M. Barz, S. Mukhametov, D. Sonntag, and J. Kuhn, "ARETT: Augmented Reality Eye
Tracking Toolkit for Head Mounted Displays," *Sensors*, vol. 21, no. 6, art. no. 2234, 2021,
doi: [10.3390/s21062234](https://doi.org/10.3390/s21062234).
