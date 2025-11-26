# Pixel Cleaner

A Python script to clean and deduplicate CSV files, specifically designed for pixel export data. This script processes large CSV files efficiently and can be deployed on Render or similar platforms.

## Features

- **Deduplication**: Removes duplicate entries based on FIRST_NAME + LAST_NAME combination
- **Phone Cleaning**: Normalizes phone numbers and filters based on DNC (Do Not Contact) flags
- **Email Extraction**: Captures the first valid email address
- **DNC Filtering**: Keeps only phone numbers with DNC flag "N" (excluding "Y" flags)
- **Large File Support**: Efficiently processes large CSV files without loading everything into memory at once

## Requirements

- Python 3.7+
- Standard library only (no external dependencies required)

## Usage

### Command Line

```bash
python pixelcleaner.py <input_csv> <output_csv>
```

### Example

```bash
python pixelcleaner.py "pixel_export of fix&flow.csv" "cleaned_output.csv"
```

## How It Works

1. **Reads the input CSV** row by row
2. **Groups records** by FIRST_NAME + LAST_NAME (case-insensitive)
3. **Collects all phone numbers** from phone-related columns (DIRECT_NUMBER, MOBILE_PHONE, PERSONAL_PHONE, SKIPTRACE_WIRELESS_NUMBERS, etc.)
4. **Matches phones with DNC flags** from corresponding DNC columns
5. **Filters phones** to keep only those with DNC flag "N" (or empty/missing DNC flag)
6. **Deduplicates** phone numbers while preserving order
7. **Extracts first email** from email-related columns
8. **Outputs cleaned CSV** with:
   - PRIMARY_PHONE and PRIMARY_EMAIL
   - Additional phones in PHONE_1, PHONE_2, etc.
   - DNC flags in PHONE_DNC_1, PHONE_DNC_2, etc.
   - All other fields (excluding original DNC columns)

## Output Format

The cleaned CSV includes:

- `FIRST_NAME`, `LAST_NAME` - Person identifiers
- `PRIMARY_PHONE` - First phone number (filtered by DNC)
- `PRIMARY_EMAIL` - First email address
- `EMAIL_1`, `EMAIL_2`, etc. - Additional emails
- `PHONE_1`, `PHONE_2`, etc. - Additional phone numbers
- `PHONE_DNC_1`, `PHONE_DNC_2`, etc. - DNC flags for corresponding phones
- All other original fields (excluding original DNC columns)

## Deployment on Render

1. Upload this script to your Render service
2. Install dependencies (none required - uses standard library)
3. Run via command line or integrate into your web service

### Example Render Service (Flask)

```python
from flask import Flask, request, send_file
import subprocess
import os

app = Flask(__name__)

@app.route('/clean', methods=['POST'])
def clean_csv():
    if 'file' not in request.files:
        return {'error': 'No file provided'}, 400
    
    input_file = request.files['file']
    input_path = f"/tmp/{input_file.filename}"
    output_path = f"/tmp/cleaned_{input_file.filename}"
    
    input_file.save(input_path)
    
    # Run the cleaner
    subprocess.run(['python', 'pixelcleaner.py', input_path, output_path])
    
    return send_file(output_path, as_attachment=True)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

## Notes

- The script processes files in chunks to handle large files efficiently
- Phone numbers are normalized (removes formatting, handles country codes)
- Only phones with DNC flag "N" (or empty) are kept in the output
- Original DNC columns (DIRECT_NUMBER_DNC, MOBILE_PHONE_DNC, etc.) are removed from output

## Performance

- Processes ~1000 rows per second
- Memory efficient - doesn't load entire file into memory
- Suitable for files with hundreds of thousands of rows

