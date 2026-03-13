# Lab 02 — Recovering Deleted Files from a Drive Image

**Source:** CompTIA Security+ CertMaster Labs — Performing Digital Forensics (Section 12.1.11)
**Environment:** KALI Linux — forensic test image `7-ntfs-undel.dd`
**Tools:** mount, tsk_recover (The Sleuth Kit / TSK)
**Technique:** Digital forensics — deleted file recovery from NTFS drive image
**Security+ Objectives:** 4.8 / 4.9
**Status:** ✅ Complete

---

## Scenario

A second investigation by the IRT into a different security incident also stalled due to lack of evidence — it was believed that the perpetrator had destroyed evidence on the affected system. A drive image from the suspect's storage device was available. The task was to attempt to recover deleted files from the NTFS drive image.

---

## Environment

| System | Role |
|--------|------|
| KALI | Forensic analysis workstation |
| 7-ntfs-undel.dd | NTFS forensic test image — deleted files present |

---

## Background: File Deletion and Recovery

When a file is deleted on most operating systems, the actual data is not immediately removed from the drive. Instead:
1. The directory entry for the file is marked as deleted
2. The disk sectors previously occupied by the file are marked as available for reuse
3. The data remains in place until those sectors are overwritten by new data

Forensic recovery tools exploit this by scanning file system metadata structures for deleted entries and attempting to recover the data before it is overwritten.

> **Important:** Recovery is not guaranteed. If sectors have been overwritten by new data, the original file content is gone. The older the deletion and the more active the system has been since, the lower the recovery probability.

---

## Setup

Extracted the NTFS test image:

```bash
cd /root/Downloads
unzip 7-undel-ntfs.zip
cd 7-undel-ntfs
ls -4
```

> Note: The zip filename is `7-undel-ntfs.zip`, the extracted directory is `7-undel-ntfs`, but the drive image inside is `7-ntfs-undel.dd`.

Target file confirmed: `7-ntfs-undel.dd`

---

## Part 1 — Mount and Inspect the Drive Image

```bash
mkdir /mnt/temp7
mount 7-ntfs-undel.dd /mnt/temp7
ls -4 /mnt/temp7
```

**Result:** Only a `System Volume Information` directory was visible — no user files present.

This indicates the suspect deleted their files. The directory listing shows nothing of investigative value from a standard mount. Recovery tools are required to access the deleted data.

---

## Part 2 — Recover Deleted Files with tsk_recover

`tsk_recover` is a TSK tool that automatically attempts to recover deleted files from a drive image into an output directory.

```bash
tsk_recover 7-ntfs-undel.dd output
ls -4 output
```

**Result:** `Files Recovered: 8`

---

## Part 3 — Identify the Recovered Files

```bash
ls -4 output
```

The recovered files were not all located in the root of the output directory — some were in subdirectories.

**Files in subdirectories:**

| File | Location |
|------|----------|
| frag3.dat | Subdirectory |
| mult2.dat | Subdirectory |

> Scored question answered correctly: frag3.dat and mult2.dat

The `frag1.dat` and `sing1.dat` files were located in the root of the output directory, not in subdirectories.

---

## Findings Summary

| Metric | Value |
|--------|-------|
| Files visible via standard mount | 0 (only System Volume Information) |
| Files recovered by tsk_recover | 8 |
| Files in subdirectories | 2 (frag3.dat, mult2.dat) |

**Next investigative steps (real-world):** The recovered files would be analysed for:
- Timestamps (created, modified, accessed — metadata forensics)
- File content (evidence of the incident)
- File type verification (extension vs actual magic bytes)
- Hash comparison against known-good or known-malicious file databases

> The test image files in this lab do not contain real content — they are placeholders to validate the recovery process. In a real investigation, the content of recovered files is the primary source of evidence.

---

## Key Takeaway

File deletion is not file destruction — it is file abandonment. Until the sectors are overwritten, the data is recoverable with the right tools. This is why forensic investigators image suspect drives immediately upon seizure (before any further system activity can overwrite deleted data) and why secure deletion tools use multi-pass overwriting rather than simple deletion. From a defensive perspective, this also means that files containing sensitive data should be securely wiped, not just deleted, before drives are decommissioned or reused.

---

*Write-up by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/) | [Back to Digital Forensics](./README.md)*
