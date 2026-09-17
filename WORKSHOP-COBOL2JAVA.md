# IBM Bob Workshop — Legacy COBOL Modernization to Java
## Case Study: Financial Calculator (COBOL to Java Migration)

### Audience
Enterprise software engineers, mainframe developers, and modernization architects working with legacy COBOL systems and Java migration projects.

### Goal of the Workshop
Demonstrate how **IBM Bob** can:
- Understand legacy COBOL financial applications
- Analyze non-obvious COBOL patterns and idioms
- Generate modern Java equivalents with proper design patterns
- Create comprehensive test suites for validation
- Build modern user interfaces for legacy business logic
- Support safe, incremental modernization strategies

We use a **COBOL Financial Calculator** consisting of three programs (COBCALC, COBLOAN, COBVALU) as a realistic example of mainframe financial software.

**Source**: IBM Debug for z/OS Sample Programs
**Reference**: https://www.ibm.com/docs/en/debug-for-zos/16.0.x?topic=mode-example-sample-cobol-program-debugging

---

## Workshop Flow Overview

1. **Understand the COBOL code** - Deep analysis of legacy patterns
2. **Convert to Java** - Modern object-oriented design
3. **Create comprehensive tests** - JUnit test suite with edge cases
4. **Build interactive UI** - Swing interface for user input
5. **Add REST API** - Microservice wrapper for the calculators
6. **Performance comparison** - Benchmark COBOL vs Java implementations
7. **Documentation generation** - API docs and architecture diagrams

Each step builds on the previous one and mirrors real-world modernization projects.

**Estimated Time:** 45–60 minutes

---

## Before You Start

> 📋 **Setup must be completed before this session.** See **[PRE-REQUISITES.md](PRE-REQUISITES.md)** for full installation instructions (Java 21, Maven, IBM Bob) for macOS, Windows, and Linux.

### Quick verification — run this now

Confirm your environment is ready before the first prompt. Open a terminal (macOS/Linux) or a new Command Prompt (Windows) and run:

**macOS / Linux**
```bash
java -version   # must show 21
mvn -version    # must show Maven 3.6+ and Java 21 on the same line
echo $JAVA_HOME # must print a non-empty path
```

**Windows**
```cmd
java -version
mvn -version
echo %JAVA_HOME%
```

Expected output (all platforms):
```
openjdk version "21.x.x" ...
Apache Maven 3.x.x ... (Java 21.x.x, ...)
<a non-empty path to your JDK 21 installation>
```

✅ If all three lines look correct, you are ready — continue to Step 1.
❌ If anything is wrong, open **[PRE-REQUISITES.md](PRE-REQUISITES.md)** and follow the troubleshooting steps for your OS before continuing.

---

## Step 1 - Understand the COBOL Code (5 minutes)

### Why this step?
Before converting legacy code, engineers must **fully understand** the business logic, data structures, and non-obvious patterns:
- COBOL-specific idioms (REDEFINES, UNSTRING, INSPECT)
- Financial calculation methods (ANNUITY, PRESENT-VALUE)
- Subprogram calling conventions (LINKAGE SECTION)
- Data type mappings (COMP, DISPLAY, PIC clauses)

This step shows that Agentic AI can perform a **deep technical analysis** of legacy code.

### Prompt
```
Analyze the COBOL financial calculator codebase and provide a comprehensive explanation 
of its functionality. Describe:

1. The overall program structure and flow
2. How COBCALC orchestrates the subprograms
3. The financial calculations performed (loan payments and present value)
4. Non-obvious COBOL patterns used (REDEFINES arrays, UNSTRING delimiters, 
   underscore placeholder technique)
5. Data type usage (COMP vs DISPLAY, PIC clauses)
6. The LINKAGE SECTION feedback mechanism

Create a Mermaid architecture diagram showing:
- Program relationships and CALL flow
- Data structures and their transformations
- Key methods and their purposes
- The calculation pipeline from input to output
```

### Expected Insights
Bob should identify:
- Hardcoded test data pattern (not production-ready)
- REDEFINES technique for array simulation
- Monthly rate conversion requirement (INTEREST / 12)
- Two-character feedback mechanism ("OK"/"NO")
- Underscore placeholder formatting technique
- Case-insensitive input handling via FUNCTION UPPER-CASE

---

## Step 2 - Convert to Java (5 minutes)

### Why this step?
Modern Java provides better maintainability, testability, and integration capabilities than legacy COBOL.

### Prompt 2a — Generate and create all files
```
Convert the COBOL financial calculator to a complete, working Maven Java project.

1. Create the following directory structure and files from scratch in this workspace:
   COBOL2JAVA/pom.xml
   COBOL2JAVA/src/main/java/com/financial/calculator/Main.java
   COBOL2JAVA/src/main/java/com/financial/calculator/CalculationResult.java
   COBOL2JAVA/src/main/java/com/financial/calculator/LoanCalculator.java
   COBOL2JAVA/src/main/java/com/financial/calculator/PresentValueCalculator.java

2. Requirements for the Java code:
   - Use BigDecimal for all financial calculations (avoid floating-point errors)
   - LoanCalculator and PresentValueCalculator as separate classes
   - CalculationResult class to replace the LINKAGE SECTION feedback mechanism
   - Preserve the original calculation logic exactly as in the COBOL source
   - Handle edge cases: zero interest, negative values, empty arrays
   - Main class that mimics COBCALC's orchestration logic

3. Requirements for pom.xml:
   - Target Java 21
   - Include JUnit 5 dependency
   - Configure maven-exec-plugin with mainClass=com.financial.calculator.Main
```

### Prompt 2b — Verify the build and run it
Send this as a follow-up immediately after Prompt 2a:
```
Now verify the project builds and runs correctly:

1. Run: mvn clean compile inside the COBOL2JAVA/ directory
2. If the build fails, read the compiler errors, fix the affected files, and recompile
   until you get BUILD SUCCESS
3. Once it compiles cleanly, run: mvn exec:java
4. Show me the full console output
```

### Key Conversion Patterns
- `PIC S9(9)V99 COMP` → `BigDecimal` with proper scale
- `PIC XX` → `String` or custom enum for status
- `FUNCTION ANNUITY` → Custom annuity formula implementation
- `FUNCTION PRESENT-VALUE` → NPV calculation with loop
- `UNSTRING ... DELIMITED BY` → `String.split()` with regex
- `INSPECT REPLACING` → `String.replace()`

### Deliverables
- `CalculationResult.java` - Result wrapper with success/failure status
- `LoanCalculator.java` - Loan payment calculator
- `PresentValueCalculator.java` - Present value calculator
- `Main.java` - Main controller
- `pom.xml` - Maven configuration

---

## Step 3 - Create Comprehensive Tests (8 minutes)

### Why this step?
Tests prevent regressions, validate conversion accuracy, and enable safe refactoring.

### Prompt
```
Create a comprehensive JUnit 5 test suite for the financial calculators and run it.

1. Create these two test files inside the COBOL2JAVA project:
   COBOL2JAVA/src/test/java/com/financial/calculator/LoanCalculatorTest.java
   COBOL2JAVA/src/test/java/com/financial/calculator/PresentValueCalculatorTest.java

2. LoanCalculatorTest.java must include 8+ test cases covering:
   - The original COBOL test data (loan=30000, rate=0.09, periods=24)
   - Zero interest rate
   - Single period loan
   - Very large loan amount
   - Negative loan amount (expect error)
   - Zero periods (expect error)
   - Correct currency formatting in the output message ($X,XXX.XX)

3. PresentValueCalculatorTest.java must include 10+ test cases covering:
   - The original COBOL test data (rate=0.12, cashFlows=[50,69,83,75,44])
   - Single cash flow
   - Negative cash flows
   - Empty cash flow array (expect error)
   - Invalid discount rate of -1 or lower (expect error)
   - Zero discount rate
   - Very large cash flows

4. After creating both files, run: mvn test inside the COBOL2JAVA/ directory
5. If any tests fail, fix either the test or the calculator code until all tests pass
6. Show me the final test results output — every test must show PASSED
```

### Expected Results
All tests pass, demonstrating:
- Conversion accuracy (matches COBOL output)
- Proper error handling
- Edge case coverage

---

## Step 4 - Build Interactive UI (10 minutes)

### Why this step?
Legacy COBOL programs use hardcoded test data. A GUI replaces those hardcoded values with real user input.

**What the two tabs do:**
- **Tab 1 — Loan Calculator**: Given a loan amount, annual interest rate, and number of monthly repayments, it calculates how much you pay each month. Example from the original COBOL: a $30,000 loan at 9% annual interest over 24 months = ~$1,370/month.
- **Tab 2 — Present Value Calculator**: Answers "what is a stream of future payments worth in today's money?" You enter a discount rate (e.g. 12%) and a list of cash flows you expect to receive each period (e.g. $50, $69, $83, $75, $44). It tells you the combined present value of all those future amounts. This mirrors exactly what `COBVALU.cbl` computed with its hardcoded values.

### Prompt 4a — Generate the GUI code

```
Create a Swing GUI (not JavaFX) for the financial calculator. Use only Swing (javax.swing)
— no JavaFX — because Swing is built into the JDK and needs no extra dependencies.

1. Create this file:
   COBOL2JAVA/src/main/java/com/financial/calculator/FinancialCalculatorGUI.java

2. The GUI must have a tabbed interface with two tabs:

   Tab 1 — "Loan Calculator":
   - Input fields: Loan Amount ($), Annual Interest Rate (%), Number of Months
   - "Calculate" button — runs the calculation
   - "Load Defaults" button — pre-fills fields with the original COBOL hardcoded values
     (loan=30000, rate=9%, periods=24) taken from COBLOAN's "30000 .09 24 " input string
   - "Clear" button — wipes all fields and the result area to blank
   - Read-only results area showing the monthly payment amount
   - Input validation: show an error dialog if any field is empty or non-numeric

   Tab 2 — "Present Value Calculator":
   - Input field: Annual Discount Rate (%)
   - A scrollable list of cash flow rows — each row has a text field for one cash flow amount
   - "Add Cash Flow" button to append a new row
   - "Remove Last" button to remove the last row
   - "Calculate" button — runs the calculation
   - "Load Defaults" button — pre-fills rate=12% and 5 cash flow rows ($50, $69, $83, $75, $44)
     taken from COBVALU's ".12 5" input and "5069837544" INPUT-BUFFER
   - "Clear" button — wipes all fields and the result area to blank
   - Read-only results area showing the present value
   - Input validation: show an error dialog if rate is empty or no cash flows are entered

3. Update COBOL2JAVA/pom.xml:
   - Remove the top-level <mainClass> from the exec-maven-plugin <configuration> block.
   - Instead, define the default mainClass as a Maven property:
       <exec.mainClass>com.financial.calculator.Main</exec.mainClass>
     inside the existing <properties> section.
   - Add a named execution inside the exec-maven-plugin <executions> block:
       <execution>
           <id>gui</id>
           <goals><goal>java</goal></goals>
           <configuration>
               <mainClass>com.financial.calculator.FinancialCalculatorGUI</mainClass>
           </configuration>
       </execution>
   This allows BOTH of these to work:
     mvn exec:java                    (runs Main — COBCALC console runner)
     mvn exec:java@gui                (runs FinancialCalculatorGUI — Swing window)

4. After creating the file and updating pom.xml:
   a. Run: mvn clean compile
      Confirm BUILD SUCCESS before continuing.
   b. Then run (do NOT kill the process — leave the window open):
        mvn exec:java@gui -Dexec.cleanupDaemonThreads=false
      The flag -Dexec.cleanupDaemonThreads=false keeps the JVM alive so the Swing
      window stays open for the user to interact with it.
   c. Confirm the console prints "FinancialCalculatorGUI: window opened." and show
      the full Maven output.
```

### Prompt 4b — Start the UI

Once Bob confirms `BUILD SUCCESS` and `FinancialCalculatorGUI: window opened.`, use this
prompt to open the window at any time during the workshop:

```
start the UI for me
```

Bob will run the correct Maven command in the background and the Swing window will appear
on your desktop. If you prefer to launch it yourself, use the command:

Run in a terminal inside the `COBOL2JAVA/` directory:
```bash
mvn exec:java@gui
```

### Deliverables
- `FinancialCalculatorGUI.java` — Swing GUI using existing calculator classes
- Updated `pom.xml` — no new dependencies needed (Swing is in the JDK)

---

## Step 5 - Add REST API (7 minutes)

### Why this step?
Modern architectures use microservices. Wrap the calculators in a REST API for integration.

### Prompt
```
Create a Spring Boot REST API for the financial calculators inside the existing
COBOL2JAVA/ Maven project.

1. Create the following files:
   COBOL2JAVA/src/main/java/com/financial/calculator/controllers/LoanController.java
   COBOL2JAVA/src/main/java/com/financial/calculator/controllers/PresentValueController.java
   COBOL2JAVA/src/main/java/com/financial/calculator/dto/LoanRequest.java
   COBOL2JAVA/src/main/java/com/financial/calculator/dto/LoanResponse.java
   COBOL2JAVA/src/main/java/com/financial/calculator/dto/PresentValueRequest.java
   COBOL2JAVA/src/main/java/com/financial/calculator/dto/PresentValueResponse.java
   COBOL2JAVA/src/main/resources/application.properties

2. Endpoints to implement:
   - POST /api/loan/calculate
     Request:  { "loanAmount": 30000, "annualRate": 0.09, "periods": 24 }
     Response: { "monthlyPayment": 1370.54, "message": "..." }

   - POST /api/presentvalue/calculate
     Request:  { "discountRate": 0.12, "cashFlows": [50, 69, 83, 75, 44] }
     Response: { "presentValue": 231.36, "message": "..." }

3. Requirements:
   - Input validation with proper HTTP status codes (400 for bad input, 200 for success)
   - JSON request/response bodies
   - CORS enabled for all origins
   - Swagger/OpenAPI documentation via springdoc-openapi
   - Set server.port=8081 in application.properties (avoids conflicts with port 8080
     which is commonly occupied on Windows by IIS or other local servers)

4. Update COBOL2JAVA/pom.xml to add Spring Boot and springdoc-openapi dependencies.
   Do not remove existing dependencies.

5. After creating all files:
   a. Run: mvn clean compile inside the COBOL2JAVA/ directory.
      Confirm BUILD SUCCESS before continuing.
   b. Then start the server — do NOT kill this process, leave it running:
        mvn spring-boot:run
      Wait until the console prints a line containing "Started FinancialCalculatorApp".
   c. Confirm the server is live by calling both endpoints with curl:
        curl -s -X POST http://localhost:8081/api/loan/calculate \
          -H "Content-Type: application/json" \
          -d '{"loanAmount":30000,"annualRate":0.09,"periods":24}' | python3 -m json.tool

        curl -s -X POST http://localhost:8081/api/presentvalue/calculate \
          -H "Content-Type: application/json" \
          -d '{"discountRate":0.12,"cashFlows":[50,69,83,75,44]}' | python3 -m json.tool
   d. Print the following URLs so the user can open them in a browser:

        Swagger UI  →  http://localhost:8081/swagger-ui.html
        OpenAPI JSON→  http://localhost:8081/v3/api-docs
```

### API Design
- **RESTful**: Proper HTTP methods and status codes
- **Validation**: Bean Validation annotations
- **Documentation**: Swagger UI at `/swagger-ui.html`
- **Testing**: Example curl commands

### Deliverables
- `LoanCalculatorController.java`
- `PresentValueCalculatorController.java`
- `LoanRequest.java` / `LoanResponse.java` DTOs
- `PresentValueRequest.java` / `PresentValueResponse.java` DTOs
- Updated `pom.xml` with Spring Boot starter web
- `application.properties` with server configuration

---

## Step 6 - Performance Comparison (Optional - 5 minutes)

### Why this step?
Validate that the Java implementation performs adequately compared to COBOL.

### Prompt
```
Create a performance benchmark comparing the Java implementation:

1. Create a benchmark harness that:
   - Runs 100,000 loan calculations
   - Runs 100,000 present value calculations
   - Measures execution time and memory usage
   - Compares BigDecimal vs double precision

2. Provide:
   - Benchmark code using JMH (Java Microbenchmark Harness)
   - Instructions to run benchmarks
   - Analysis of results
   - Recommendations for optimization if needed

3. Test scenarios:
   - Single-threaded performance
   - Multi-threaded performance (parallel streams)
   - Memory allocation patterns
```

### Expected Insights
- BigDecimal overhead vs double
- JVM warmup effects
- Optimization opportunities

---

## Step 7 - Documentation Generation (Optional - 5 minutes)

### Why this step?
Comprehensive documentation supports long-term maintenance and onboarding.

### Prompt
```
Generate comprehensive documentation for the Java financial calculator:

1. JavaDoc comments for all public methods
2. Architecture diagram showing:
   - Class relationships
   - Data flow
   - API endpoints (if REST API was created)
3. User guide with:
   - How to build and run
   - API usage examples
   - Configuration options
4. Migration guide documenting:
   - COBOL to Java mapping decisions
   - Known differences
   - Testing strategy

Generate using Maven site plugin and provide instructions to view.
```

### Documentation Deliverables
- JavaDoc HTML
- Architecture diagrams (Mermaid/PlantUML)
- User guide (Markdown)
- API documentation (Swagger)

---

## Summary & Key Takeaways

### What You've Accomplished
1. ✅ Analyzed legacy COBOL financial software
2. ✅ Converted to modern Java with proper design patterns
3. ✅ Created comprehensive test suite (26 tests)
4. ✅ Built interactive user interface
5. ✅ Wrapped in REST API for microservices
6. ✅ Benchmarked performance (Optional)
7. ✅ Generated complete documentation (Optional)

### Key Modernization Patterns Learned
- **Data Type Mapping**: COBOL COMP → Java BigDecimal
- **Subprogram Conversion**: CALL/LINKAGE → Method calls/Return objects
- **Financial Precision**: Avoiding floating-point errors
- **Test-Driven Migration**: Validate conversion accuracy
- **UI Modernization**: From batch to interactive
- **API-First Design**: Enable integration with modern systems

### Real-World Applications
This workshop demonstrates patterns applicable to:
- Banking and financial services modernization
- Insurance policy calculation systems
- Payroll and benefits processing
- Inventory and pricing systems
- Any COBOL system with complex business logic

### Next Steps
- Apply these patterns to your own COBOL codebases
- Extend with additional features (reporting, audit trails)
- Integrate with modern data stores (PostgreSQL, MongoDB)
- Deploy to cloud platforms (AWS, Azure, IBM Cloud)
- Implement CI/CD pipelines for continuous modernization

---

## Appendix: Quick Reference

### Maven Commands
```bash
mvn clean compile                                        # Compile the project
mvn test                                                 # Run all tests
mvn exec:java                                            # Run the console app (Main)
mvn exec:java@gui -Dexec.cleanupDaemonThreads=false      # Launch the Swing GUI
mvn spring-boot:run                                      # Run Spring Boot API (if created)
mvn package                                              # Build JAR file
```

### Expected Test Results
- **Total Tests**: 26
- **Loan Calculator Tests**: 12 (all passing)
- **Present Value Tests**: 14 (all passing)
- **Execution Time**: < 1 second

### Key Files Created
```
COBOL2JAVA-Lab/                          ← repo root
├── COBCALC.cbl                          ← original COBOL source
├── COBLOAN.cbl                          ← original COBOL source
├── COBVALU.cbl                          ← original COBOL source
├── WORKSHOP-COBOL2JAVA.md
├── PRE-REQUISITES.md
├── README.md
└── COBOL2JAVA/                          ← generated by Bob during the workshop
    ├── pom.xml
    └── src/
        ├── main/java/com/financial/calculator/
        │   ├── Main.java
        │   ├── CalculationResult.java
        │   ├── LoanCalculator.java
        │   ├── PresentValueCalculator.java
        │   ├── FinancialCalculatorGUI.java         (Step 4)
        │   ├── controllers/                        (Step 5)
        │   └── dto/                                (Step 5)
        └── test/java/com/financial/calculator/
            ├── LoanCalculatorTest.java
            └── PresentValueCalculatorTest.java
```

---

**Workshop Version**: 1.1
**Last Updated**: 2026-09-16
**Estimated Duration**: 30-45 minutes
**Difficulty Level**: Intermediate
