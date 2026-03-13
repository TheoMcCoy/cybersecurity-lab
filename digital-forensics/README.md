# Digital Forensics

**Source:** CompTIA Security+ CertMaster Labs — Performing Digital Forensics (Section 12.1.11)
**Status:** ✅ Complete

---

## Overview

This folder contains write-ups from hands-on digital forensics labs completed as part of the CompTIA Security+ CertMaster Lab pathway. Labs cover three core forensic techniques: drive image partition analysis and hidden partition discovery, deleted file recovery from NTFS images, and file carving from corrupted drive images.

---

## Labs in This Folder

| # | Lab | Technique | Status |
|---|-----|-----------|--------|
| 01 | [Forensic Drive Image Analysis: Hidden Partition Discovery](./01-forensic-drive-image-hidden-partition.md) | fdisk, testdisk, fiwalk, fsstat, mmls, fls, istat (TSK) | ✅ Complete |
| 02 | [Recovering Deleted Files from a Drive Image](./02-recovering-deleted-files.md) | tsk_recover, NTFS undelete | ✅ Complete |
| 03 | [File Carving from a Corrupted Drive Image](./03-file-carving.md) | testdisk carving, boot sector rebuild, FAT16 recovery | ✅ Complete |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| fdisk | Standard partition table display |
| testdisk | Advanced partition analysis + file carving |
| fiwalk | Drive image walk — file system and file reporting |
| fsstat | File system statistics (requires `-f` type, `-o` offset) |
| mmls | Partition layout with sector offsets |
| fls | File listing at a given partition offset |
| istat | Inode metadata inspection |
| tsk_recover | Automated deleted file recovery |
| mount / losetup | Mounting drive images as virtual devices |

---

## Key Concepts Covered

- Forensic drive image acquisition and analysis methodology
- MBR partition structure — primary vs extended vs logical
- Hidden partition detection — fdisk vs testdisk discrepancy
- TSK (The Sleuth Kit) tool chain for layered forensic analysis
- Inode structure and metadata (size, permissions, timestamps — not filename or content)
- File deletion mechanics — why deleted files are recoverable until sectors are overwritten
- File carving — signature-based recovery when metadata is destroyed
- Boot sector corruption and in-memory rebuild via testdisk

---

*Write-ups by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/) | [Back to Main Repo](../../README.md)*
