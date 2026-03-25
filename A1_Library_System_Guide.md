# 📚 Assignment 1: Library Borrowing & Fine Management System — Blueprint Guide

---

## Overview

At first glance, this assignment might look like a lot to handle. But if we break it down piece by piece, it's basically just:

1. Reading a file
2. Saving that data into objects and arrays
3. Doing some basic math for fines
4. Printing everything out in a neatly formatted report

This guide walks through the full blueprint — step by step.

---

## Step 0: The Blueprint — UML Class Diagram

Before writing any complex logic, we need to build our **data containers**. The assignment provides a UML diagram with four main classes. You need to create each one with its **private variables**, **constructor**, **getters**, **setters**, and a `toString()` method.

### Classes to Build

| Class           | Private Fields                                                   | Required Methods                                                                                            |
| --------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `Book`          | `isbn`, `title`, `author`, `publishYear`                         | constructors, getters/setters, `toString()`                                                                 |
| `Member`        | `memberID`, `firstName`, `lastName`, `birthYear`                 | constructors, getters/setters, `toString()`                                                                 |
| `BorrowRecord`  | `recordNo`, `book`, `member`, `fineCode`, `severity`, `baseFine` | constructors, `calculateFinalFine()`, `toString()`                                                          |
| `LibrarySystem` | All arrays as fields                                             | `addBook()`, `addMember()`, `addFineRule()`, `generateBorrow()`, `printBorrow()`, `printBorrowsPerMember()` |

> 💡 **Tip:** `BorrowRecord` is the most complex class because it _contains_ a `Book` object and a `Member` object. This is called **composition** — one of the key OOP concepts being tested here.

> ⚠️ **Important:** The assignment requires you to **organize your code in separate methods**. Don't put everything inside `main()` — each operation (`addBook`, `addMember`, etc.) should be its own method.

---

## Step 1: Setting Up the Arrays (The Storage)

The system requires **five arrays** in total. Their sizes come directly from the first three lines of `input.txt`.

### Reading the Input File Header

Looking at the actual `input.txt`:

```
4 4       ← Line 1: booksLength=4, membersLength=4
3         ← Line 2: rulesLength=3  (number of fine rule types)
12        ← Line 3: recordsLength=12 (number of borrow records)
```

```java
int booksLength   = input.nextInt(); // 1st number on line 1
int membersLength = input.nextInt(); // 2nd number on line 1
int rulesLength   = input.nextInt(); // number on line 2
int recordsLength = input.nextInt(); // number on line 3
```

### The Five Arrays

```java
// 1. Stores all registered Book objects
Book[] books = new Book[booksLength];

// 2. Stores all registered Member objects
Member[] members = new Member[membersLength];

// 3. Ragged array — stores fine amounts per rule type and severity level
double[][] fineRules = new double[rulesLength][];

// 4. Stores all issued BorrowRecord objects
BorrowRecord[] records = new BorrowRecord[recordsLength];

// 5. Fixed array of fine type names — used to find the correct row in fineRules
String[] fineCodes = { "LateReturn", "DamagedBook", "LostBook" };
```

> 💡 The `fineCodes` array is the **key** to the ragged array. When you get a fine code like `"LateReturn"` from the file, you search `fineCodes` for its index (0), then use that index to look up the correct row in `fineRules`.

---

### The Tricky Part: The Ragged Array

The assignment requires a **2D Ragged Array** for the fine rules: `double[][] fineRules`.

**What is a ragged array?**

- A standard 2D array is a perfect grid (every row has the same number of columns).
- A **ragged array** has rows of _different_ lengths.

From `input.txt`, we can see exactly how this looks in practice:

| Fine Type               | Level 1 | Level 2 | Level 3 |
| ----------------------- | ------- | ------- | ------- |
| `LateReturn` (index 0)  | 10.0    | 20.0    | 50.0    |
| `DamagedBook` (index 1) | 100.0   | 300.0   | —       |
| `LostBook` (index 2)    | 500.0   | —       | —       |

**How to handle it — in three stages:**

```java
// Stage 1: Set the number of ROWS only. Columns are undefined for now.
double[][] fineRules = new double[rulesLength][];

// Stage 2: Inside your addFineRule() method, when reading "AddFineRule" from the file:
// e.g. "AddFineRule LateReturn 3 10 20 50"
String fineCode = input.next();    // "LateReturn"
int levels      = input.nextInt(); // 3

// Find which row this fine code belongs to using fineCodes[]
int index = -1;
for (int i = 0; i < fineCodes.length; i++) {
    if (fineCodes[i].equals(fineCode)) {
        index = i;
        break;
    }
}

// Create a 1D array of exactly the right size and fill it
double[] amounts = new double[levels];
for (int i = 0; i < levels; i++) {
    amounts[i] = input.nextDouble();
}

// Stage 3: Slot that 1D array into the correct row of the 2D array.
fineRules[index] = amounts;
```

This is the key insight: each **row** of the ragged array is just a regular 1D array assigned after reading how many levels it has.

---

## Step 2: Reading the Input (File I/O)

We read from `input.txt` sequentially. The clean structure for this is a `while` loop with a `switch` statement, where each `case` calls a dedicated method.

> ⚠️ Always wrap file reading in a `try-catch` block!

```java
try {
    File inputFile = new File("input.txt");
    Scanner input  = new Scanner(inputFile);

    // Read the three header lines first
    int booksLength   = input.nextInt();
    int membersLength = input.nextInt();
    int rulesLength   = input.nextInt();
    int recordsLength = input.nextInt();

    // Initialize all arrays...

    while (input.hasNext()) {
        String command = input.next();

        switch (command) {
            case "AddBook":
                addBook(input, books);
                break;
            case "AddMember":
                addMember(input, members);
                break;
            case "AddFineRule":
                addFineRule(input, fineRules, fineCodes);
                break;
            case "IssueBorrow":
                generateBorrow(input, records, books, members, fineRules, fineCodes);
                break;
            case "Quit":
                printBorrowsPerMember(writer, records, members);
                break;
        }
    }
} catch (FileNotFoundException e) {
    System.out.println("Error: input.txt not found.");
}
```

### How `IssueBorrow` / `generateBorrow()` Works

The command `IssueBorrow 9780134685991 2023123456 LateReturn 2` requires four searches:

1. **Search `books[]`** by ISBN → find the `Book` object.
2. **Search `members[]`** by member ID → find the `Member` object.
3. **Search `fineCodes[]`** by fine code name → get the row index for `fineRules`.
4. **Look up `fineRules[index][severity - 1]`** → get the `baseFine` amount.

> ⚠️ Note: severity levels in the file are **1-based** (1, 2, 3), but arrays are **0-based**. So severity 2 maps to `fineRules[index][1]`. Always subtract 1 when indexing.

Then create a new `BorrowRecord` and store it in `records[]`. The record number is generated automatically — just use a counter that increments each time.

---

## Step 3: Calculating Fines (The Core Logic)

Inside your `BorrowRecord` class, the `calculateFinalFine()` method applies three rules:

| Rule   | Condition                  | Action                                      |
| ------ | -------------------------- | ------------------------------------------- |
| Rule 1 | Always                     | Start with `baseFine` from the ragged array |
| Rule 2 | Member age < 18            | Add **+100.0 SAR**                          |
| Rule 3 | Book published before 2000 | Add **+50.0 SAR**                           |

> Use **2026** as the current year when calculating age.

```java
public double calculateFinalFine() {
    double totalFine = baseFine; // Rule 1: Base fine from fineRules array

    // Rule 2: Underage member surcharge
    int age = 2026 - member.getBirthYear();
    if (age < 18) {
        totalFine += 100.0;
    }

    // Rule 3: Old book surcharge
    if (book.getPublishYear() < 2000) {
        totalFine += 50.0;
    }

    return totalFine;
}
```

### Verifying With Real Data

Cross-checking against the actual `LibraryDB.txt` output confirms the logic:

| Record | Member      | Birth Year | Age | Book Pub. Year | Base Fine | +Underage | +Old Book | **Total** |
| ------ | ----------- | ---------- | --- | -------------- | --------- | --------- | --------- | --------- |
| 1      | Ahmed Saleh | 2011       | 15  | 2019           | 20.0      | +100      | —         | **120.0** |
| 2      | Ahmed Saleh | 2011       | 15  | 1998           | 300.0     | +100      | +50       | **450.0** |
| 3      | Sara Ali    | 2006       | 20  | 2006           | 100.0     | —         | —         | **100.0** |
| 4      | Omar Khaled | 1995       | 31  | 1999           | 500.0     | —         | +50       | **550.0** |
| 8      | Omar Khaled | 1995       | 31  | 2019           | 20.0      | —         | —         | **20.0**  |

---

## Step 4: Writing the Output (The Report)

> ⚠️ **Critical requirement:** Output **must** be written to `LibraryDB.txt`. Do **not** use `System.out.println()`. The file should be **overwritten** on every run.

```java
PrintWriter writer = new PrintWriter("LibraryDB.txt");
// ... write all records and the summary ...
writer.close(); // Always close the writer or the file may not save!
```

### 4.1 Borrow Record Format

Each borrow record printed by `printBorrow()` must follow this exact structure (separator line included):

```
=============== Borrow Record Information ===============
Record No= 1
Fine Code= LateReturn
Severity= 2
Base Fine= 20.0

Book Information
ISBN= 9780134685991
Title= JavaProgramming
Author= Deitel
Publish Year= 2019

Member Information
ID= 2023123456
Name= Ahmed Saleh
Birth Year= 2011

Total Fine= 120.0
```

> 💡 Notice the **blank lines** between sections — these are part of the required format. Match them exactly.

### 4.2 Summary Report Format (`printBorrowsPerMember()`)

Printed after all borrow records when `Quit` is encountered:

```
-------- Total Borrow(s) Per Member --------
Member ID     Name           Total Borrows
2023123456    Ahmed Saleh    4
2023987456    Sara Ali       3
2022456789    Omar Khaled    2
2022765432    Laila Hassan   3
```

### Formatting Secret: `printf`

Use `writer.printf()` to align the summary table columns perfectly:

```java
// Header
writer.printf("%-14s%-15s%s\n", "Member ID", "Name", "Total Borrows");

// Each member row (loop through members[])
writer.printf("%-14s%-15s%d\n", member.getMemberID(), fullName, borrowCount);
```

**How the format specifiers work:**

| Specifier | Meaning                                                        |
| --------- | -------------------------------------------------------------- |
| `%-14s`   | `String`, left-justified, padded to **14** characters wide     |
| `%-15s`   | `String`, left-justified, padded to **15** characters wide     |
| `%d`      | An `int` (whole number — for borrow count)                     |
| `%.1f`    | A `double` with 1 decimal place (for fine amounts like `20.0`) |
| `\n`      | New line                                                       |

**How to count borrows per member:** Loop through `records[]` and for each record, check if `record.getMember().getMemberID().equals(member.getMemberID())`. Count the matches.

---

### 4.3 Output Formatting Rules (From the PDF)

- The output file must be **overwritten** each time the program runs.
- Borrow records must appear in the **same order** as the `IssueBorrow` commands in `input.txt`.
- The summary report appears **after** all borrow records.
- **No extra text or debug output** should appear in the file.
- Numeric fine values must display with a decimal point (e.g., `20.0`, not `20`).

---

## Step 5: Submission Requirements

> ⚠️ **Read carefully — wrong file name = lost marks.**

- Submit **one `.zip` file** on Blackboard containing your Java file.
- File name format: `SectionID_A1.java`.
- Your **name, ID, and section number** must appear as comments at the top of every file.
- **Due Date:** Saturday, 28/3/2026 at 11:00 PM — no late submissions accepted.

---

## ✅ Final Checklist

Before submitting, go through each item:

- [ ] All class fields are **private** with **public** getters and setters
- [ ] All three classes (`Book`, `Member`, `BorrowRecord`) have a `toString()` method
- [ ] `BorrowRecord` correctly holds a `Book` object and a `Member` object (composition)
- [ ] Logic is separated into dedicated methods: `addBook()`, `addMember()`, `addFineRule()`, `generateBorrow()`, `printBorrow()`, `printBorrowsPerMember()`
- [ ] `fineCodes[]` string array is declared and used to map fine names to `fineRules` row indices
- [ ] Severity level is converted from **1-based** (file) to **0-based** (array index) when looking up `baseFine`
- [ ] The `fineRules` ragged array rows are assigned dynamically based on the number of severity levels per rule
- [ ] File reading is wrapped in a `try-catch` block
- [ ] Output is written to **`LibraryDB.txt`** using `PrintWriter` — **not** `System.out.println()`
- [ ] `writer.close()` is called after all output is written
- [ ] Borrow record output format (separator line, blank lines, labels) matches exactly
- [ ] Summary table uses `printf` with correct column widths
- [ ] Fine amounts display with a decimal point (e.g., `500.0`)
- [ ] Your **file name** matches the required `SectionID_A1.java` format exactly
- [ ] Your **name, ID, and section** are included as comments at the top of the file
- [ ] Code is documented with comments throughout

---

_Good luck, and happy coding! 🚀_
