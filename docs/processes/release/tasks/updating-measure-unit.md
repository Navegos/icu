---
layout: default
title: Updating MeasureUnit with new CLDR data
parent: Release & Milestone Tasks
grand_parent: Contributors
nav_order: 120
---

<!--
© 2020 and later: Unicode, Inc. and others.
License & terms of use: http://www.unicode.org/copyright.html
-->

# Updating MeasureUnit with new CLDR data

{: .no_toc }

## Contents

{: .no_toc .text-delta }

1. TOC
{:toc}

---

This document explains how to update the C++ and Java version of the `MeasureUnit`
class with new CLDR data.

This document applies to ICU 77 and later.
For older versions see updating-measure-unit-old.md

Make sure `DRAFT_VERSION_SET` at top of
`./icu4j/main/common_tests/src/test/java/com/ibm/icu/dev/test/format/MeasureUnitGeneratorTest.java`
is set correctly. \
These are the ICU versions that have draft methods.

The code is generated by running `MeasureUnitGeneratorTest.java` unit tests, which writes
generated code to various file.

1. With **maven** (command line):
   - Change folder to `{icuRoot}/icu4j`
   - run `mvn install -DskipTests -DskipITs`
   - run `mvn install -q -Dtest=MeasureUnitGeneratorTest -DgenerateMeasureUnitUpdate -f main/common_tests`

2. Within **Eclipse**:
   - Open `MeasureUnitGeneratorTest.java`, find the `generateUnitTestsUpdate` methods
    and run it by clicking on the green play button on menu bar. \
    Choose "JUnit Test" if asked. \
    This will not generate the update, but it will run the test and create a "Run Configuration". \
    Open it (Main menu -- "Run" -- "Run Configurations"), select the one named
    `MeasureUnitGeneratorTest.generateUnitTestsUpdate`, go to the "Arguments" tab and add
    `-DgenerateMeasureUnitUpdate` to the "VM Arguments" text area.

Both methods will generate files with in `icu4j/main/common_tests/target/` folder. \
The file names and the logging to the standard output will guide you.

It currently looks something like this:
```
Copy the generated code fragments from / to
    /some/absolute/path/icu4j/main/common_tests/target/MeasureUnit.java \
    /some/absolute/path/icu4j/main/core/src/main/java/com/ibm/icu/util/MeasureUnit.java

Copy the generated code fragments from / to
    /some/absolute/path/icu4j/main/common_tests/target/MeasureUnitCompatibilityTest.java \
    /some/absolute/path/icu4j/main/common_tests/src/test/java/com/ibm/icu/dev/test/format/MeasureUnitCompatibilityTest.java

Copy the generated code fragments from / to
    /some/absolute/path/icu4j/main/common_tests/target/measunit.h \
    /some/absolute/path/icu4c/source/i18n/unicode/measunit.h

Copy the generated code fragments from / to
    /some/absolute/path/icu4j/main/common_tests/target/measunit.cpp \
    /some/absolute/path/icu4c/source/i18n/measunit.cpp

Copy the generated code fragments from / to
    /some/absolute/path/icu4j/main/common_tests/target/measfmttest.cpp \
    /some/absolute/path/icu4c/source/test/intltest/measfmttest.cpp

Copy the generated code fragments from / to
    /some/absolute/path/icu4j/main/common_tests/target/MeasureUnitGeneratorTest.java \
    /some/absolute/path/icu4j/main/common_tests/src/test/java/com/ibm/icu/dev/test/format/MeasureUnitGeneratorTest.java
```

Some kind of diff tool or editor (for example `vi -d`) work nicely.

Look for line containing `// Start generated ...` and `// End generated ...`
These lines exist in both the original files, and the generated one. \
Replace all the generated code in between with the contents of the clipboard.

If the generated code has no `// Start` ... `// End ...` pair then the new
code should be appended at some fixed place (details below).

* **`MeasureUnit.java`:** replace range.
* **`MeasureUnitCompatibilityTest.java`:** append the new generated method at the end. \
  It is named something like `TestCompatible<version>()`. \
  Don't add it if it already exists.
* **`measunit.h`:** replace range.
* **`measunit.cpp`:** replace range.
* **`measfmttest.cpp`:** append the new generated method after the last
  `MeasureFormatTest::TestCompatible<version>()` method. \
  Don't add it if it already exists. \
  WARNING: here you should add the method in two places. The method proper, with code,
  as generated, and the declaration in the class definition.
* **`MeasureUnitGeneratorTest.java`:** append the new pairs of measure + version at
  the end of the `JAVA_VERSIONS` structure. \
  Don't add them if they already exist.

## Run tests for both `icu4c` and `icu4j`

## Updating `units.txt` and `unitConstants`

The standard `ldml2icu` process is used to update ICU's resource files (see
[`cldr-icu-readme.txt`](https://github.com/unicode-org/icu/blob/main/icu4c/source/data/cldr-icu-readme.txt)).
CLDR's units.xml defines conversion rates in terms of some constants defined in
`unitConstants`.

For efficiency and simplicity, ICU does not read `unitConstants` from the
resource file. If any new constants are added, some code changes would be
needed. This would be caught by `testUnitConstantFreshness` unit test in
`units_test.cpp`.

They are hard-coded:

* Java: `UnitConverter.java` has the constant names in
  `UnitConverter.Factor.addEntity()` and constant values in
  `UnitConverter.Factor.getConversionRate()`.
* C++: `units_converter.cpp` has the constant names in
  `addSingleFactorConstant()`, with the constant values in `double
  constantsValues[]` in the `units_converter.h` header file.