# PES-VCS Lab Report

**Course:** Operating Systems Lab
**Assignment:** Building PES-VCS — A Version Control System from Scratch
**Student Name:** Atharv Mittal
**SRN:** PES1UG24CS091
**Platform:** Ubuntu 22.04
**Repository Name:** PES1UG24CS091-pes-vcs

---

# Table of Contents

1. Objective
2. Environment Setup
3. Build Commands
4. Phase 1 – Object Storage Foundation
5. Phase 2 – Tree Objects
6. Phase 3 – Index (Staging Area)
7. Phase 4 – Commits and History
8. Phase 5 – Branching and Checkout (Analysis)
9. Phase 6 – Garbage Collection (Analysis)
10. Submission Checklist
11. GitHub Repository Link

---

# 1. Objective

The objective of this assignment was to build a lightweight local version control system inspired by Git. The system supports object storage, staging files, creating commits, and viewing commit history while demonstrating core filesystem concepts such as hashing, atomic writes, directory trees, references, and linked history.

---

# 2. Environment Setup

## Prerequisites

```bash
sudo apt update
sudo apt install -y gcc build-essential libssl-dev
```

## Build Commands

```bash
make
make all
make clean
```

---

# 3. Files Implemented

| File     | Functions Implemented                   |
| -------- | --------------------------------------- |
| object.c | object_write(), object_read()           |
| tree.c   | tree_from_index()                       |
| index.c  | index_load(), index_save(), index_add() |
| commit.c | commit_create()                         |

---

# 4. Phase 1 – Object Storage Foundation

## Concepts Covered

* Content-addressable storage
* SHA-256 hashing
* Deduplication
* Atomic writes
* Integrity verification

## Implemented Functions

* `object_write()`
* `object_read()`

## Testing Commands

```bash
make test_objects
./test_objects
```

## 📸 Screenshot 1A – Test Output

**Insert screenshot of:**

```bash
./test_objects
```

> Paste Screenshot Here

---

## 📸 Screenshot 1B – Object Store Structure

**Insert screenshot of:**

```bash
find .pes/objects -type f
```

> Paste Screenshot Here

---

# 5. Phase 2 – Tree Objects

## Concepts Covered

* Directory representation
* Tree serialization
* Recursive filesystem structure
* File modes

## Implemented Function

* `tree_from_index()`

## Testing Commands

```bash
make test_tree
./test_tree
```

## 📸 Screenshot 2A – Tree Test Output

**Insert screenshot of:**

```bash
./test_tree
```

> Paste Screenshot Here

---

## 📸 Screenshot 2B – Raw Tree Object Format

**Insert screenshot of:**

```bash
find .pes/objects -type f
xxd .pes/objects/XX/YYYY... | head -20
```

(Use actual object path)

> Paste Screenshot Here

---

# 6. Phase 3 – Index (Staging Area)

## Concepts Covered

* File metadata tracking
* Staging area
* Atomic index writes
* Change detection

## Implemented Functions

* `index_load()`
* `index_save()`
* `index_add()`

## Testing Commands

```bash
./pes init
echo hello > file1.txt
echo world > file2.txt
./pes add file1.txt file2.txt
./pes status
cat .pes/index
```

## 📸 Screenshot 3A – Init + Add + Status

**Insert screenshot of:**

```bash
./pes init
./pes add file1.txt file2.txt
./pes status
```

> Paste Screenshot Here

---

## 📸 Screenshot 3B – Index File Contents

**Insert screenshot of:**

```bash
cat .pes/index
```

> Paste Screenshot Here

---

# 7. Phase 4 – Commits and History

## Concepts Covered

* Linked commit history
* References
* HEAD pointer
* Snapshot commits

## Implemented Function

* `commit_create()`

## Testing Commands

```bash
./pes init

echo Hello > hello.txt
./pes add hello.txt
./pes commit -m "Initial commit"

echo World >> hello.txt
./pes add hello.txt
./pes commit -m "Add world"

echo Bye > bye.txt
./pes add bye.txt
./pes commit -m "Add farewell"

./pes log
```

## 📸 Screenshot 4A – Commit History

**Insert screenshot of:**

```bash
./pes log
```

> Paste Screenshot Here

---

## 📸 Screenshot 4B – PES Repository Growth

**Insert screenshot of:**

```bash
find .pes -type f | sort
```

> Paste Screenshot Here

---

## 📸 Screenshot 4C – HEAD and Branch Reference

**Insert screenshot of:**

```bash
cat .pes/refs/heads/main
cat .pes/HEAD
```

> Paste Screenshot Here

---

# 8. Phase 5 – Branching and Checkout (Analysis)

## Q5.1 How would `pes checkout <branch>` work?

A branch is a file inside `.pes/refs/heads/` that stores the latest commit hash. To implement `pes checkout <branch>`, the system must first verify that the branch file exists. Then `.pes/HEAD` should be updated to point to that branch reference. After that, the commit hash stored in the branch file is read, its tree object is loaded, and the working directory files are rewritten to match that snapshot.

The complex part is safely updating the working directory. Existing files may need deletion, modification, or creation. Uncommitted changes must be detected first to avoid data loss.

---

## Q5.2 How to detect dirty working directory conflicts?

The index stores file paths, timestamps, sizes, and hashes. Before checkout, compare each tracked file in the working directory against the index metadata. If size or modification time differs, the file may be changed.

Then compare the target branch tree with the current branch tree. If a file has local changes and the target branch also changes that same file, checkout must refuse and ask the user to commit or stash changes first.

---

## Q5.3 What is Detached HEAD?

Detached HEAD means `.pes/HEAD` stores a commit hash directly instead of a branch name. New commits created in this state are valid commits, but no branch points to them automatically.

If the user switches away, those commits may become unreachable. Recovery is possible by creating a new branch pointing to that commit hash or manually updating a reference file.

---

# 9. Phase 6 – Garbage Collection (Analysis)

## Q6.1 How to find unreachable objects?

Start from all branch references inside `.pes/refs/heads/`. Every referenced commit is reachable. From each commit, recursively mark:

* parent commits
* tree objects
* blobs referenced by trees

Use a **hash set** to store visited object hashes efficiently.

After traversal, scan `.pes/objects/`. Any object not in the reachable set is unreachable and can be deleted.

For 100,000 commits and 50 branches, roughly all commits plus their trees and blobs may be visited, depending on shared objects. The number could be several hundred thousand total objects.

---

## Q6.2 Why is concurrent garbage collection dangerous?

Suppose a commit operation writes new tree/blob objects first, but has not yet updated `.pes/refs/heads/main`. At the same time, garbage collection scans references and does not see those new objects as reachable yet. It may delete them before the commit finishes.

Then the branch updates to a commit that points to missing objects, corrupting the repository.

Real Git avoids this using locks, temporary files, reflogs, and careful coordination so GC does not delete recently created or in-progress objects.

---

# 10. Submission Checklist

## Required Screenshots

* [ ] Screenshot 1A – `./test_objects`
* [ ] Screenshot 1B – `find .pes/objects -type f`
* [ ] Screenshot 2A – `./test_tree`
* [ ] Screenshot 2B – `xxd` raw tree object
* [ ] Screenshot 3A – `pes init → add → status`
* [ ] Screenshot 3B – `cat .pes/index`
* [ ] Screenshot 4A – `pes log`
* [ ] Screenshot 4B – `find .pes -type f | sort`
* [ ] Screenshot 4C – `cat .pes/refs/heads/main` + `cat .pes/HEAD`
* [ ] Final – `make test-integration`

---

## Required Code Files

* [ ] object.c
* [ ] tree.c
* [ ] index.c
* [ ] commit.c

---

## Commit History Requirement

Minimum **5 commits per phase** with clear messages.

Check using:

```bash
git log --oneline --graph
```

---

# 11. GitHub Repository Link

Paste your public GitHub repository link here:

**Repository:** `https://github.com/YOUR_USERNAME/PES1UG24CS091-pes-vcs`

---

# Final Note

This project successfully demonstrates the internal working principles of version control systems such as Git using low-level filesystem operations and data structures.
