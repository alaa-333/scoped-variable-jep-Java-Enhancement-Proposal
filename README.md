<div align="center">

<img src="https://img.icons8.com/fluency/96/java-coffee-cup-logo.png" width="80"/>

# Scoped Mutable Variables

### Java Enhancement Proposal

<br/>

[![JEP Status](https://img.shields.io/badge/JEP_Status-Draft-E8E8E8?style=for-the-badge&labelColor=2B2B2B)](/)
[![Java Version](https://img.shields.io/badge/Target-Java_22+-E8E8E8?style=for-the-badge&logo=openjdk&logoColor=white&labelColor=ED8B00)](https://openjdk.org/)
[![Proposal Type](https://img.shields.io/badge/Type-Feature-E8E8E8?style=for-the-badge&labelColor=5865F2)](/)

<br/>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

*A compile-time mechanism to confine variable mutability within lexical scopes,*
*preventing unintended side effects in complex control flows.*

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

<br/>

[Overview](#-overview) · [Motivation](#-motivation) · [Specification](#-specification) · [Examples](#-examples) · [Implementation](#-implementation) · [References](#-references)

<br/>

</div>

---

<br/>

## ▎Document Information

<table>
<tr>
<td><b>JEP Number</b></td>
<td><i>To Be Assigned</i></td>
<td><b>Author</b></td>
<td>Alaa Mohamed</td>
</tr>
<tr>
<td><b>Status</b></td>
<td>Draft</td>
<td><b>Target Release</b></td>
<td>Java 22+</td>
</tr>
<tr>
<td><b>Type</b></td>
<td>Feature Proposal</td>
<td><b>Component</b></td>
<td>Language Specification</td>
</tr>
</table>

<br/>

---

## ▎Overview

This proposal introduces a new variable declaration modifier — **`scoped`** — enabling variables to be mutable exclusively within a specific lexical scope. Modifications made within nested blocks (`if`, `for`, `while`) do not propagate to outer scopes.

<br/>

<div align="center">
<table>
<tr>
<td align="center" width="280">
<br/>
<img src="https://img.icons8.com/fluency/48/shield.png" width="36"/>
<br/><br/>
<b>Prevents Side Effects</b>
<br/>
<sub>Isolates mutations within<br/>defined boundaries</sub>
<br/><br/>
</td>
<td align="center" width="280">
<br/>
<img src="https://img.icons8.com/fluency/48/code.png" width="36"/>
<br/><br/>
<b>Clean Syntax</b>
<br/>
<sub>Single keyword, minimal<br/>learning curve</sub>
<br/><br/>
</td>
<td align="center" width="280">
<br/>
<img src="https://img.icons8.com/fluency/48/checked.png" width="36"/>
<br/><br/>
<b>Fully Compatible</b>
<br/>
<sub>No impact on existing<br/>Java codebases</sub>
<br/><br/>
</td>
</tr>
</table>
</div>

<br/>

---

## ▎Motivation

### The Problem

In large-scale Java applications, a common source of bugs is the **unintended mutation** of variables declared in outer scopes but modified within control-flow blocks. This pattern leads to subtle, hard-to-trace defects.

<br/>

<div align="center">
<table>
<tr>
<th width="420">❌ &nbsp; Current Approach</th>
<th width="420">✓ &nbsp; Proposed Solution</th>
</tr>
<tr>
<td>

```java
// Verbose manual preservation
int originalCount = count;

if (condition) {
    count = 50;
    process(count);
}

count = originalCount; // Easy to forget
```

</td>
<td>

```java
// Clean, intentional scoping
scoped int count = 100;

if (condition) {
    count = 50;
    process(count);
}
// count remains 100 — guaranteed
```

</td>
</tr>
<tr>
<td align="center"><sub>Error-prone • Verbose • Intent unclear</sub></td>
<td align="center"><sub>Safe • Concise • Self-documenting</sub></td>
</tr>
</table>
</div>

<br/>

---

## ▎Goals & Non-Goals

<br/>

<table>
<tr>
<td width="50%" valign="top">

### Goals

<br/>

&nbsp;&nbsp;&nbsp;&nbsp;◆&nbsp;&nbsp; Introduce concise `scoped` modifier syntax

&nbsp;&nbsp;&nbsp;&nbsp;◆&nbsp;&nbsp; Restrict mutability to defined code blocks

&nbsp;&nbsp;&nbsp;&nbsp;◆&nbsp;&nbsp; Preserve outer scope values automatically

&nbsp;&nbsp;&nbsp;&nbsp;◆&nbsp;&nbsp; Integrate with existing type system

&nbsp;&nbsp;&nbsp;&nbsp;◆&nbsp;&nbsp; Maintain full backward compatibility

<br/>

</td>
<td width="50%" valign="top">

### Non-Goals

<br/>

&nbsp;&nbsp;&nbsp;&nbsp;○&nbsp;&nbsp; Altering semantics of `final` keyword

&nbsp;&nbsp;&nbsp;&nbsp;○&nbsp;&nbsp; Enforcing pure immutability

&nbsp;&nbsp;&nbsp;&nbsp;○&nbsp;&nbsp; Adopting functional programming paradigm

&nbsp;&nbsp;&nbsp;&nbsp;○&nbsp;&nbsp; Introducing runtime scope barriers

&nbsp;&nbsp;&nbsp;&nbsp;○&nbsp;&nbsp; Modifying JVM specifications

<br/>

</td>
</tr>
</table>

<br/>

---

## ▎Specification

### Syntax Definition

The `scoped` modifier precedes variable declarations and supports both explicit typing and type inference:

```java
scoped int counter = 0;        // Explicit type declaration
scoped var message = "init";   // Type inference with var
```

<br/>

### Semantic Behavior

<div align="center">

```
                    ┌─────────────────────────────────────┐
                    │   scoped int value = 100;           │
                    │                                     │
                    │   ┌─────────────────────────────┐   │
                    │   │  if (condition) {           │   │
                    │   │      value = 50;  ◄─────────┼───┼─── Shadow created
                    │   │      // value == 50         │   │
                    │   │  }  ◄────────────────────────┼───┼─── Shadow discarded
                    │   └─────────────────────────────┘   │
                    │                                     │
                    │   // value == 100  ◄────────────────┼─── Original preserved
                    └─────────────────────────────────────┘
```

</div>

<br/>

<table>
<tr>
<td width="40"><b>1</b></td>
<td><b>Declaration</b> — A scoped variable creates a new symbol with an initial value</td>
</tr>
<tr>
<td><b>2</b></td>
<td><b>Shadow Creation</b> — Nested blocks automatically create a shadowed instance for local reassignment</td>
</tr>
<tr>
<td><b>3</b></td>
<td><b>Block Exit</b> — Upon exiting the block, the shadowed instance is discarded</td>
</tr>
<tr>
<td><b>4</b></td>
<td><b>Preservation</b> — The outer variable retains its original value unchanged</td>
</tr>
</table>

<br/>

---

## ▎Examples

### Basic Control Flow

```java
scoped int count = 100;

if (shouldModify) {
    count = 50;
    System.out.println(count);     // Output: 50
}

System.out.println(count);         // Output: 100
```

<br/>

### Loop Constructs

```java
scoped int accumulator = 0;

for (int i = 1; i <= 5; i++) {
    accumulator += i;
    System.out.println("Loop: " + accumulator);
}
// Loop: 1, Loop: 3, Loop: 6, Loop: 10, Loop: 15

System.out.println("Final: " + accumulator);
// Final: 0
```

<br/>

### Nested Scope Hierarchy

```java
scoped String state = "INITIAL";

if (conditionA) {
    state = "PROCESSING_A";
    
    if (conditionB) {
        state = "PROCESSING_B";
        System.out.println(state);     // Output: PROCESSING_B
    }
    
    System.out.println(state);         // Output: PROCESSING_A
}

System.out.println(state);             // Output: INITIAL
```

<br/>

---

## ▎Implementation

### Architecture Overview

<div align="center">

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            COMPILATION PIPELINE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────┐      ┌─────────────┐      ┌─────────────┐                │
│   │             │      │             │      │             │                │
│   │   PARSER    │ ───▶ │   SYMBOL    │ ───▶ │  BYTECODE   │                │
│   │             │      │   TABLE     │      │  GENERATOR  │                │
│   └─────────────┘      └─────────────┘      └─────────────┘                │
│         │                    │                    │                        │
│         ▼                    ▼                    ▼                        │
│   ┌─────────────┐      ┌─────────────┐      ┌─────────────┐                │
│   │  Recognize  │      │  Generate   │      │    Emit     │                │
│   │   scoped    │      │   unique    │      │  synthetic  │                │
│   │  modifier   │      │   symbols   │      │  variables  │                │
│   └─────────────┘      └─────────────┘      └─────────────┘                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────────────────────┐
                    │   JVM — No Modifications Required   │
                    └─────────────────────────────────────┘
```

</div>

<br/>

### Component Changes

| Component | Modification | Complexity |
|:----------|:-------------|:----------:|
| **Lexer** | Add `scoped` as contextual keyword | Low |
| **Parser** | Extend local variable grammar rules | Low |
| **Symbol Table** | Generate unique identifiers for shadow variables | Medium |
| **Type Checker** | Validate scoped variable usage patterns | Medium |
| **Bytecode Generator** | Emit synthetic local variables for each scope | Medium |
| **JVM** | *None required* | — |

<br/>

---

## ▎Alternatives Considered

<br/>

| Approach | Description | Assessment |
|:---------|:------------|:-----------|
| **Manual Duplication** | Copy variables before blocks, restore after | Verbose, error-prone, intent unclear |
| **Linting Rules** | Static analysis to detect in-block mutations | Non-preventive, requires external tooling |
| **Wrapper Objects** | Encapsulate values in immutable containers | Adds complexity and runtime overhead |
| **Final + New Variables** | Declare new variables within each scope | Clutters code, naming challenges |

<br/>

---

## ▎Compatibility

<table>
<tr>
<td width="160"><b>Source Compatibility</b></td>
<td>Fully compatible — <code>scoped</code> is a new contextual keyword</td>
</tr>
<tr>
<td><b>Binary Compatibility</b></td>
<td>Fully compatible — no changes to class file format</td>
</tr>
<tr>
<td><b>Behavioral Compatibility</b></td>
<td>Fully compatible — existing programs unchanged</td>
</tr>
</table>

<br/>

---

## ▎Future Directions

<br/>

<table>
<tr>
<td align="center" width="80">
<img src="https://img.icons8.com/fluency/32/switch.png" width="24"/>
</td>
<td>
<b>Switch Expressions</b><br/>
<sub>Extend scoped variable support to switch expressions and pattern matching constructs</sub>
</td>
</tr>
<tr>
<td align="center">
<img src="https://img.icons8.com/fluency/32/lambda.png" width="24"/>
</td>
<td>
<b>Lambda Integration</b><br/>
<sub>Enable scoped semantics for variables captured by lambda expressions</sub>
</td>
</tr>
<tr>
<td align="center">
<img src="https://img.icons8.com/fluency/32/diamond.png" width="24"/>
</td>
<td>
<b>Project Valhalla</b><br/>
<sub>Explore interactions with value types and primitive classes</sub>
</td>
</tr>
<tr>
<td align="center">
<img src="https://img.icons8.com/fluency/32/lock.png" width="24"/>
</td>
<td>
<b>Scoped Immutability</b><br/>
<sub>Consider <code>scoped final</code> for read-only access within nested scopes</sub>
</td>
</tr>
</table>

<br/>

---

## ▎References

<br/>

| Document | Description |
|:---------|:------------|
| [JLS §6.3 — Scope of Declarations](https://docs.oracle.com/javase/specs/jls/se21/html/jls-6.html) | Java Language Specification on variable scoping |
| [JEP 286 — Local-Variable Type Inference](https://openjdk.org/jeps/286) | Introduction of `var` keyword in Java 10 |
| [JEP 361 — Switch Expressions](https://openjdk.org/jeps/361) | Pattern for scoped expression evaluation |

<br/>

---

<br/>

<div align="center">

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

<br/>

<sub>

**Java Enhancement Proposal** · Draft Document

Authored by **Alaa Mohamed**

</sub>

<br/>

[![Document](https://img.shields.io/badge/Document-JEP_Draft-2B2B2B?style=flat-square)](/)
[![Version](https://img.shields.io/badge/Version-1.0-2B2B2B?style=flat-square)](/)
[![License](https://img.shields.io/badge/License-CC_BY_4.0-2B2B2B?style=flat-square)](/)

<br/>

<sub>This document follows the standard format for Java Enhancement Proposals as defined by the OpenJDK community.</sub>

</div>
