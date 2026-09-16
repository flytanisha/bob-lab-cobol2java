# IBM Bob Workshop — Legacy COBOL Modernization to Java

A hands-on lab that shows how **IBM Bob** understands decades-old mainframe COBOL, converts it to clean object-oriented Java, writes tests, builds a GUI, and wraps it in a REST API — all through conversational prompts.

**Estimated time:** 45–60 minutes  
**Difficulty:** Intermediate  
**Audience:** Enterprise engineers, mainframe developers, modernization architects

---

## What is in this repository

```
COBOL2JAVA-Lab/
├── COBCALC.cbl              ← Main COBOL controller program
├── COBLOAN.cbl              ← Loan payment calculator subprogram
├── COBVALU.cbl              ← Present value calculator subprogram
├── WORKSHOP-COBOL2JAVA.md   ← Step-by-step workshop guide (start here)
├── PRE-REQUISITES.md        ← Setup guide — complete this before the workshop
└── README.md                ← This file
```

The `COBOL2JAVA/` project folder is created by Bob during the workshop. It does not exist yet.

---

## The COBOL source code

The three COBOL programs are IBM sample programs from the [Debug for z/OS documentation](https://www.ibm.com/docs/en/debug-for-zos/16.0.x?topic=mode-example-sample-cobol-program-debugging). They form a small financial calculator suite typical of mainframe batch applications.

### COBCALC.cbl — The controller

Loops through a hardcoded list of commands (`LOAN`, `PVALUE`, `END`) and delegates to the appropriate subprogram via `CALL`. This is the mainline program.

```cobol
BUFFER-DATA.
    10  FILLER  PIC X(10)  VALUE "LOAN".
    10  FILLER  PIC X(10)  VALUE "PVALUE".
    10  FILLER  PIC X(10)  VALUE "pvalue".
    10  FILLER  PIC X(10)  VALUE "END".

EVALUATE FUNCTION UPPER-CASE(INPUT-1)
  WHEN "LOAN"    PERFORM CALCULATE-LOAN
  WHEN "PVALUE"  PERFORM CALCULATE-VALUE
  WHEN "END"     MOVE "END" TO INPUT-1
  WHEN OTHER     DISPLAY "Invalid input: " INPUT-1
END-EVALUATE.
```

**Notable patterns:**
- Input commands are hardcoded in a `REDEFINES` array — no real user input
- `FUNCTION UPPER-CASE` normalises case so `"pvalue"` and `"PVALUE"` both work
- Subprograms return a two-character `CALL-FEEDBACK` (`"OK"` / `"NO"`) via the `LINKAGE SECTION`

---

### COBLOAN.cbl — Loan payment calculator

Calculates the fixed monthly repayment on a loan. All inputs are hardcoded as a single string and split at runtime.

```cobol
MOVE "30000 .09 24 " TO INPUT-1.
UNSTRING INPUT-1 DELIMITED BY ALL " "
  INTO LOAN-AMOUNT-IN INTEREST-IN NO-OF-PERIODS-IN.

COMPUTE PAYMENT = LOAN-AMOUNT *
    FUNCTION ANNUITY((INTEREST / 12) NO-OF-PERIODS).
```

**What it computes:** Monthly repayment on a $30,000 loan at 9% annual interest over 24 months ≈ **$1,370.54/month**

**Notable patterns:**
- `PIC S9(9)V99 USAGE COMP` — signed packed-decimal binary field (maps to `BigDecimal` in Java)
- `PIC $$$$,$$$,$$9.99` — currency display picture with leading `$` and comma formatting
- `FUNCTION ANNUITY` — COBOL intrinsic annuity formula, has no direct Java equivalent and must be reimplemented
- Underscores used as space placeholders in string building (`INSPECT REPLACING ALL "_" BY SPACES`)

---

### COBVALU.cbl — Present value calculator

Calculates what a series of future cash flows is worth in today's money, given a discount rate. Cash flows are packed into a 10-character string and read two characters at a time.

```cobol
MOVE ".12 5 " TO INPUT-1.
INPUT-BUFFER  PIC X(10) VALUE "5069837544".
BUFFER-ARRAY  REDEFINES INPUT-BUFFER OCCURS 5 TIMES PIC XX.

COMPUTE PAYMENT =
    FUNCTION PRESENT-VALUE(INTEREST VALUE-AMOUNT(ALL)).
```

**What it computes:** Present value of cash flows $50, $69, $83, $75, $44 at a 12% discount rate ≈ **$231.36**

**Notable patterns:**
- `REDEFINES` used to treat a single string as a 5-element array — classic COBOL array simulation
- `FUNCTION PRESENT-VALUE` — COBOL intrinsic NPV function, must be reimplemented as a loop in Java
- `OCCURS 99` declared for `VALUE-AMOUNT` but only 5 slots used — typical COBOL over-allocation

---

## What the workshop produces

By the end, Bob will have generated a complete Java project inside `COBOL2JAVA/`:

| Step | What gets created |
|------|-------------------|
| Step 1 | Architecture analysis + Mermaid diagram |
| Step 2 | `LoanCalculator.java`, `PresentValueCalculator.java`, `CalculationResult.java`, `Main.java`, `pom.xml` |
| Step 3 | `LoanCalculatorTest.java`, `PresentValueCalculatorTest.java` — 26 passing JUnit 5 tests |
| Step 4 | `FinancialCalculatorGUI.java` — Swing desktop app with two calculator tabs |
| Step 5 | Spring Boot REST API with Swagger UI at `http://localhost:8081/swagger-ui.html` |

---

## How to run the workshop

1. Complete **[PRE-REQUISITES.md](PRE-REQUISITES.md)** before the session
2. Follow **[WORKSHOP-COBOL2JAVA.md](WORKSHOP-COBOL2JAVA.md)** step by step — paste each prompt into Bob and let it do the work
