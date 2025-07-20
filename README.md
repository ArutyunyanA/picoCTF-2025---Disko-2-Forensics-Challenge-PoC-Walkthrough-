### Disko 2 PicoCTF Forensics Challenge

## Main task
"Can you find the flag in this disk image? The right one is Linux! One wrong step and it’s all gone!"

# Step 1 - Unpacking the image
"Download the file and unpack correctly "
```bash
gunzip disko-2.dd.gz
```
# Step 2 - Partition table analysis
"Using the fdisk utility, determine the offset of the required partition"
```bash
fdisk -lu disko-2.dd
048,51200,0x83,-,0,32,33,3,80,13
53248,65536,0x0B,-,3,80,14,7,100,29
0,0,0x00,-,0,0,0,0,0,0
0,0,0x00,-,0,0,0,0,0,0
```
"The output showed that the section we are interested in starts from sector 2048 and has a size of 51200 sectors."

# Step 3 - Extracting a partition from an image
"Cut out the required part from the .dd file using dd"
```bash
dd if=disko-2.dd of=part1.img bs=512 skip=2048 count=51200
51200+0 records in
51200+0 records out
26214400 bytes transferred in 0.420929 secs (62277493 bytes/sec)

```

-> bs=512 — size of one sector

-> skip=2048 — skip first 2048 sectors

-> count=51200 — read 51200 sectors

"As a result, we get the file part1.img, which is one of the sections"

# Step 4 - Search for flag
"Inside part1.img there were many strings simulating flags. We used strings and grep to pull out candidates"
```bash
strings part1.img | grep picoCTF
picoCTF{4_P4Rt_1t_i5_a93c3ba0}
```
"Since many of the flags were fakes, we need to sort the flags by the number of repetitions and find the unique one."

```bash
strings part1.img | grep picoCTF | sort | uniq -c | sort -n
```

"Another way as a bonus you can use my script in step 4"
```python

import re                  
import sys                 
from collections import Counter  

def extract_flags(filepath):

    try:
        with open(filepath, 'rb') as f:   # Open the file in binary mode
            data = f.read()               # Read the entire file
    except FileNotFoundError:
        print(f"File not found: {filepath}")  # If the file is not found, we report it and exit
        sys.exit(1)

    # Decode the contents into a string. Use latin1 and ignore errors,
    # so as not to crash on binary data, where there may be incorrect characters.
    text = data.decode('latin1', errors='ignore')

    # Regular expression searches for substrings of the form picoCTF{...}
    # Lazy quantification .*? — captures the minimum number of characters up to }
    flags = re.findall(r'picoCTF\{.*?\}', text)

    return flags  # Return a list of all flags found

def main():
    # Check that exactly 1 argument (file name) is passed when running the script
    if len(sys.argv) != 2:
        print("Usage: python3 extract_flags.py <filename>")
        sys.exit(1)

    filepath = sys.argv[1]     ## Get file name from command line arguments

    flags = extract_flags(filepath)  # Extract the list of flags from the file

    if not flags:  # If no flags found
        print("No picoCTF flags found.")
        sys.exit(0)

    counts = Counter(flags)  # Count how many times each flag occurs

    print("\nUnique picoCTF flags:\n")
    # We output only those flags that have occurred exactly once - this is usually the real flag
    for flag, count in counts.items():
        if count == 1:
            print(flag)

if __name__ == "__main__":
    main()

```
