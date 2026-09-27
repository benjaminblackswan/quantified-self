# Pipeline design




# Dashboard Update Procedure

Update Frequency : Yearly

1. Combine game recordings for each playthrough PL### and upload to Youtube channel
2. Update the Playthrough Log.xlsx
3. Move game recording files to Temp Holding drive (D), with Playthrough ID as the file name.
4. Run the following python code using PyCharm to extract gameplay information to the gaming.daily table.

```
import pyodbc
import cv2
from pathlib import Path
import re
from datetime import datetime

# --- SQL Server connection (on-prem) using Windows Authentication ---
server = 'localhost'
database = 'Ben'
driver = '{ODBC Driver 17 for SQL Server}'

conn = pyodbc.connect(
    f"DRIVER={driver};SERVER={server};DATABASE={database};Trusted_Connection=yes;"
)
cursor = conn.cursor()

# --- Ensure schema exists ---
cursor.execute("""
IF NOT EXISTS (SELECT * FROM sys.schemas WHERE name = 'gaming')
    EXEC('CREATE SCHEMA gaming')
""")
conn.commit()

# --- Ensure table exists ---
cursor.execute("""
IF OBJECT_ID(N'gaming.daily', N'U') IS NULL
BEGIN
    CREATE TABLE gaming.daily (
        PlayDate DATE,
        PlayTime VARCHAR(8),
        PlayID VARCHAR(5),
        DurationSecond DECIMAL(10,3)
    )
END
""")
conn.commit()

# --- Video folder path ---
video_folder_path = Path(r"D:\Gaming")

video_files = sorted([
    f for f in video_folder_path.rglob("*")
    if f.suffix.lower() in ['.mp4', '.mov', '.avi', '.mkv']
])

# Counters for logging
inserted_count = 0
skipped_count = 0

for video in video_files:
    cap = cv2.VideoCapture(str(video))

    try:
        fps = cap.get(cv2.CAP_PROP_FPS)
        frame_count = cap.get(cv2.CAP_PROP_FRAME_COUNT)

        duration_seconds = round(frame_count / fps, 3) if fps else 0

        # --- Extract date and time from filename ---
        # Format: YYYY-MM-DD_HH.MM.SS GM
        date_time_match = re.search(
            r'(\d{4}-\d{2}-\d{2})_(\d{2}\.\d{2}\.\d{2})',
            video.stem
        )

        if date_time_match:
            date_part, time_part = date_time_match.groups()

            # Convert HH.MM.SS -> HH-MM-SS
            play_time = time_part.replace(".", ":")

            # Convert date string to DATE
            play_date = datetime.strptime(
                date_part,
                "%Y-%m-%d"
            ).date()

        else:
            # Fallback if pattern not found
            now = datetime.now()
            play_date = now.date()
            play_time = now.strftime("%H-%M-%S")

        # --- Use first 5 characters of parent folder as PlayID ---
        play_id = video.parent.name[:5]

        # --- Insert if not exists ---
        cursor.execute("""
            INSERT INTO gaming.daily
                (PlayDate, PlayTime, PlayID, DurationSecond)
            SELECT ?, ?, ?, ?
            WHERE NOT EXISTS (
                SELECT 1
                FROM gaming.daily
                WHERE PlayDate = ?
                  AND PlayTime = ?
                  AND PlayID = ?
                  AND DurationSecond = ?
            )
        """,
        play_date,
        play_time,
        play_id,
        duration_seconds,
        play_date,
        play_time,
        play_id,
        duration_seconds)

        if cursor.rowcount == 0:
            skipped_count += 1
            print(f"Skipped duplicate {play_id}/{video.name}")
        else:
            inserted_count += 1
            print(
                f"Inserted {play_id}/{video.name} -> "
                f"{play_date} {play_time} -> {duration_seconds} sec"
            )

    finally:
        cap.release()

conn.commit()
cursor.close()
conn.close()

print(
    f"✅ Processing complete: "
    f"{inserted_count} rows inserted, "
    f"{skipped_count} rows skipped."
)
```
