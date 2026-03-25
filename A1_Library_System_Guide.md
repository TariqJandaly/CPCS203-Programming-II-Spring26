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

Before writing any complex logic, we need to build our **data containers**. The assignment provides a UML diagram with four main classes. You need to create each one with its **private variables**, **constructor**, **getters**, and **setters**.

### Classes to Build

| Class           | Private Fields                                                                 |
| --------------- | ------------------------------------------------------------------------------ |
| `Book`          | `isbn`, `title`, `author`, `publishYear`                                       |
| `Member`        | `memberID`, `firstName`, `lastName`, `birthYear`                               |
| `BorrowRecord`  | `Book` object, `Member` object, `recordNo`, `fineCode`, `severity`, `baseFine` |
| `LibrarySystem` | Contains the `main` method and all arrays                                      |

> 💡 **Tip:** `BorrowRecord` is the most complex class because it _contains_ a `Book` object and a `Member` object. This is called **composition** — one of the key OOP concepts being tested here.

---

## Step 1: Setting Up the Arrays (The Storage)

The system reads books, members, fine rules, and borrow records from `input.txt` and stores them in arrays.

The **very first line** of `input.txt` gives us the exact sizes we need:

```java
// Read the lengths from the first line of the file
int booksLength   = input.nextInt(); // 1st number
int membersLength = input.nextInt(); // 2nd number
int rulesLength   = input.nextInt(); // 3rd number
int recordsLength = input.nextInt(); // 4th number

// Initialize the arrays
Book[]         books   = new Book[booksLength];
Member[]       members = new Member[membersLength];
BorrowRecord[] records = new BorrowRecord[recordsLength];
```

---

### The Tricky Part: The Ragged Array

The assignment requires a **2D Ragged Array** for the fine rules: `double[][] fineRules`.

**What is a ragged array?**

- A standard 2D array is a perfect grid (like a checkerboard — every row has the same number of columns).
- A **ragged array** has rows of _different_ lengths. For example, `LateReturn` might have 3 severity levels, while `LostBook` has only 1.

**How to handle it — in three stages:**

```java
// Stage 1: Set the number of ROWS only. Leave column size undefined for now.
double[][] fineRules = new double[rulesLength][];

// Stage 2: Later, when reading the "AddFineRule" command from the file:
int levels = input.nextInt();          // How many severity levels does THIS rule have?
double[] amounts = new double[levels]; // Create a 1D array of exactly that size

// ... loop here to fill the amounts array with values from the file ...

// Stage 3: Slot that 1D array into the correct row of the 2D array.
fineRules[index] = amounts;
```

This is the key insight: each **row** of the ragged array is just a regular 1D array that you assign after reading how many levels it has.

---

## Step 2: Reading the Input (File I/O)

We read from a file called `input.txt`. Commands are processed **sequentially** from top to bottom. A clean way to structure this is a `while` loop combined with a `switch` statement.

> ⚠️ Always wrap file reading in a `try-catch` block!

```java
File inputFile = new File("input.txt");
Scanner input  = new Scanner(inputFile);

while (input.hasNext()) {
    String command = input.next();

    switch (command) {
        case "AddBook":
            // Read the next 4 tokens → construct a Book object → store in books[]
            break;

        case "AddMember":
            // Read the next 4 tokens → construct a Member object → store in members[]
            break;

        case "AddFineRule":
            // Apply the ragged array logic from Step 1
            break;

        case "IssueBorrow":
            // Find the matching Book and Member by ID
            // Link them together inside a new BorrowRecord
            break;

        case "Quit":
            // Trigger the report-writing logic
            break;
    }
}
```

### How `IssueBorrow` Works

When you hit `IssueBorrow`, the file will give you a book ISBN and a member ID. Your job is to:

1. **Search** through your `books[]` array to find the matching `Book` object.
2. **Search** through your `members[]` array to find the matching `Member` object.
3. **Create** a new `BorrowRecord` linking both of them together, then store it in `records[]`.

---

## Step 3: Calculating Fines (The Core Logic)

Inside your `BorrowRecord` class, create a method called `calculateFinalFine()`. This method applies three rules in sequence:

| Rule   | Condition                       | Action                                                                    |
| ------ | ------------------------------- | ------------------------------------------------------------------------- |
| Rule 1 | Always                          | Start with `baseFine` (looked up from the ragged array by severity level) |
| Rule 2 | Member age < 18                 | Add **100.0 SAR**                                                         |
| Rule 3 | Book published before year 2000 | Add **50.0 SAR**                                                          |

> Use **2026** as the current year when calculating a member's age.

```java
public double calculateFinalFine() {
    double totalFine = baseFine; // Rule 1: Start with the base fine

    // Rule 2: Check if the member is underage
    int age = 2026 - member.getBirthYear();
    if (age < 18) {
        totalFine += 100.0;
    }

    // Rule 3: Check if the book is old (published before 2000)
    if (book.getPublishYear() < 2000) {
        totalFine += 50.0;
    }

    return totalFine;
}
```

---

## Step 4: Writing the Output (The Report)

> ⚠️ **Critical requirement:** Output **must** be written to `LibraryDB.txt` — not printed to the console with `System.out.println()`.

Use a `PrintWriter` to write to the file:

```java
PrintWriter writer = new PrintWriter("LibraryDB.txt");
```

---

### Formatting Secret: `printf`

When the `Quit` command is reached, you print a summary report. To make columns align perfectly (as shown in the assignment PDF), use `writer.printf()`.

```java
// Print the table header
writer.printf("%-14s%-15s%s\n", "Member ID", "Name", "Total Borrows");

// Inside your loop, print each member's row
writer.printf("%-14s%-15s%d\n", memberID, fullName, borrowCount);
```

**How the format specifiers work:**

| Specifier      | Meaning                                                            |
| -------------- | ------------------------------------------------------------------ |
| `%-14s`        | A `String`, left-justified (`-`), padded to **14** characters wide |
| `%-15s`        | A `String`, left-justified, padded to **15** characters wide       |
| `%d`           | An `int` (whole number)                                            |
| `%f` or `%.2f` | A `double`, optionally rounded to 2 decimal places                 |
| `\n`           | New line                                                           |

This creates invisible columns so everything aligns without manually counting spaces.

---

## ✅ Final Checklist

Before submitting, go through each item:

- [ ] All class fields are **private** with **public** getters and setters
- [ ] `BorrowRecord` correctly holds a `Book` object and a `Member` object (composition)
- [ ] The `fineRules` ragged array rows are assigned dynamically based on the number of severity levels read from the file
- [ ] File reading is wrapped in a `try-catch` block
- [ ] Output is written to **`LibraryDB.txt`** using `PrintWriter` (not `System.out.println`)
- [ ] `printf` format specifiers are used to align report columns correctly
- [ ] Your **main class name** matches the required `SectionID_A1` format exactly
- [ ] Code includes **comments** explaining your logic

---

_Good luck, and happy coding! 🚀_
