# Lab 01 — Forensic Drive Image Analysis: Hidden Partition Discovery

**Source:** CompTIA Security+ CertMaster Labs — Performing Digital Forensics (Section 12.1.11)
**Environment:** KALI Linux — forensic test image `ext-part-test-2.dd`
**Tools:** fdisk, testdisk, fiwalk, fsstat, mmls, fls, istat (The Sleuth Kit / TSK)
**Technique:** Digital forensics — partition analysis, hidden partition discovery, inode inspection
**Security+ Objectives:** 4.8 / 4.9
**Status:** ✅ Complete

---

## Scenario

An investigation by an IRT (Incident Response Team) into a recent intrusion stalled due to lack of evidence — the original system had already been reconstituted. A raw drive image of the targeted system was available for forensic analysis. The task was to perform post-incident forensic analysis to discover evidence left behind by the intruder, starting with a full partition analysis of the drive image.

---

## Environment

| System | Role |
|--------|------|
| KALI | Forensic analysis workstation |
| Student-Resources-L25.ISO | ISO media containing forensic test image files |
| ext-part-test-2.dd | Forensic test image — Extended partition layout |

---

## Background: Forensic Drive Images

A **forensic drive image** (`.dd` file) is a bit-for-bit copy of a storage device captured using a tool like `dd` or Guymager. Because the image is exact, all partition structures, file system metadata, deleted files, and hidden data are preserved exactly as they existed on the original drive — even if the original system has been wiped or reconstituted.

The forensic test image used in this lab (`ext-part-test-2.dd`) is sourced from the **Digital Forensics Tool Testing image repository** at `http://dftt.sourceforge.net` — a repository of 14 forensic images used to test and validate forensic analysis tools.

---

## Setup

Mounted the Student-Resources-L25.ISO DVD and copied forensic test files to the working directory:

```bash
ls /media/cdrom/
cp /media/cdrom/* /root/Downloads/
cd /root/Downloads
unzip 1-extend-part.zip
cd 1-extend-part
ls -4
```

Target file confirmed: `ext-part-test-2.dd`

---

## Part 1 — Partition Analysis with fdisk

```bash
fdisk -l ext-part-test-2.dd
```

**fdisk output summary:**

| Finding | Value |
|---------|-------|
| 4th partition type | Extended |
| Formattable partitions displayed | **5** |
| Logical drives in extended partition | 2 (sizes: 52353 and 50337 sectors) |

> Scored question answered correctly: 5 formattable partitions

**MBR partition context:** MBR-based drives support a maximum of 4 primary partitions. The 4th primary partition can be converted to an **Extended partition**, which can then be subdivided into multiple **logical drives** — allowing more than 4 formatted volumes. The extended partition itself is not directly formattable.

**Unaccounted sectors:** The extended partition showed more sectors than the two logical drives accounted for. Possible explanations:
- A hidden logical drive within the extended partition
- Unused space not allocated to any logical drive

> Scored questions answered correctly: **A hidden logical drive** and **Unused space not allocated to a logical drive**

---

## Part 2 — Deeper Analysis with testdisk

```bash
testdisk -l ext-part-test-2.dd
```

**testdisk result:** **7 numbered entries** — one more than fdisk reported.

This extra entry is the hidden partition that fdisk could not detect. testdisk uses deeper partition table analysis and can identify entries that are obscured or non-standard.

---

## Part 3 — Drive Image Analysis with fiwalk

```bash
fiwalk ext-part-test-2.dd | less
```

fiwalk is a TSK-based tool that walks through a drive image and reports on every file system and file it can identify. In this image, each partition contained a text file named after the partition — providing a breadcrumb trail confirming partition boundaries.

---

## Part 4 — File System Statistics with fsstat and mmls

### Step 1 — Initial fsstat Attempt

```bash
fsstat ext-part-test-2.dd
```

Result: **Error — file system type was not determined.** The fiwalk output showed the file system type as "fat16" — fsstat requires the type to be specified explicitly for this image.

```bash
fsstat -f fat16 ext-part-test-2.dd
```

Result: **Error — invalid magic value.** The initial partition table is corrupted and cannot be automatically interpreted.

### Step 2 — Determine Partition Offset with mmls

```bash
mmls ext-part-test-2.dd
```

mmls is a TSK tool that displays the layout of all partitions including their sector offsets. The **Start** column shows the sector offset for each division.

**Key finding:** The hidden extended volume was identified as **line 14** in the mmls output.

### Step 3 — fsstat with Sector Offset

```bash
fsstat -f fat16 ext-part-test-2.dd -o 262143
```

Using the sector offset from mmls, fsstat successfully returned file system details for the hidden partition.

---

## Part 5 — File Listing and Inode Inspection (TSK)

### Step 1 — List Files with fls

```bash
fls -f fat16 ext-part-test-2.dd -o 262143
```

Result: A file named **second-3.txt** was visible, along with `$`-prefixed system-hidden volume management components.

### Step 2 — Inspect Inodes with istat

Cycled through inodes 1–5 in the hidden partition:

```bash
istat -f fat16 ext-part-test-2.dd -o 262143 1
# Error: Metadata address is too small for image

istat -f fat16 ext-part-test-2.dd -o 262143 2
# Reveals root directory information

istat -f fat16 ext-part-test-2.dd -o 262143 3
# File in root of drive named second-3.txt

istat -f fat16 ext-part-test-2.dd -o 262143 4
# SECOND-3.txt with timestamps — file was present before partition was hidden
# Deleted or corrupted — unable to be restored

istat -f fat16 ext-part-test-2.dd -o 262143 5
# Invalid metadata address — end of available inode data
```

**Finding from inode 4:** This file had timestamps while other inode entries did not. This suggests it was present on the drive **before** it was converted into a hidden partition — likely a file that was in use before the attacker created the hidden volume. It was subsequently deleted or corrupted and is unrecoverable from this test image.

> An inode is a file system metadata structure storing file size, owner, permissions, and timestamps — but not the file name (stored in the directory entry) or the file content (stored in data blocks).

---

## Part 6 — Mount and Explore the Hidden Partition

```bash
losetup --partscan --find --show ext-part-test-2.dd
# Output: /dev/loop0 — virtual device created

mkdir /mnt/p6
mount /dev/loop0p7 /mnt/p6
# /dev/loop0p7 references the 7th partition table item (6th logical volume in extended partition)

cd /mnt/p6
ls -4
```

**Contents:** `second-3.txt` visible. `SECOND-3.txt` was not present — it had been deleted and overwritten, confirming it is unrecoverable.

---

## TSK Tool Reference

| Tool | Purpose |
|------|---------|
| `fdisk` | Display standard partition table entries |
| `testdisk` | Advanced partition table analysis — finds hidden entries |
| `fiwalk` | Walk drive image, report all file systems and files |
| `fsstat` | File system statistics — requires `-f` for type, `-o` for offset |
| `mmls` | Display all partition layout including sector offsets |
| `fls` | List files in a partition at a given offset |
| `istat` | Display inode metadata for individual files |

---

## Key Takeaway

Hidden partitions are a real attacker technique — an extended partition can contain additional logical volumes that standard tools like fdisk may not report. In this lab, testdisk found 7 entries where fdisk only found 5. The critical forensic discipline is **tool diversity**: no single tool sees everything. Using fdisk → testdisk → fiwalk → fsstat → mmls → fls → istat in sequence revealed progressively more detail until the hidden partition and its contents were fully accessible.

---

*Write-up by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/) | [Back to Digital Forensics](./README.md)*
