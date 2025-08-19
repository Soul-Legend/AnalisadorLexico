# Lexical & Syntactic Analyzer Generator

This project provides a generator for lexical and syntactic analyzers, implemented in Python with a CustomTkinter GUI.

The lexical analysis component accepts token definitions as regular expressions, constructs the corresponding finite automata (NFA and DFA), and uses the optimized DFA to tokenize input source code. It supports two DFA construction methods: Thompson's algorithm and the direct method via followpos computation.

The syntactic analysis component accepts a context-free grammar, computes the necessary sets (First, Follow, LR(0) items), builds an SLR parsing table, and uses it to parse a stream of tokens. An integrated mode connects both phases.

## Features

*   **Lexical Analysis (Lexer Generator):**
    *   **Regular Expression Input:** Define token specifications using regular expressions.
    *   **Dual DFA Construction Methods:**
        *   **Thompson's Algorithm:** RE → NFA → DFA → Minimized DFA.
        *   **Followpos (Direct Method):** RE → Augmented Syntax Tree → Followpos Table → DFA → Minimized DFA.
    *   **Automata Visualization:** Textual and graphical (via Graphviz) representations of generated automata.
    *   **Symbol Table:** Manages static definitions and dynamic symbols found during tokenization.
*   **Syntactic Analysis (Parser Generator):**
    *   **Context-Free Grammar Input:** Define language syntax using production rules.
    *   **SLR(1) Parser Generation:** Computes First sets, Follow sets, the LR(0) canonical collection, and the final SLR parsing table.
    *   **Parse Tracing:** Outputs a step-by-step trace of the stack, input, and actions taken by the parser.
*   **User Interface:**
    *   A graphical interface for defining specifications, controlling the generation process, and viewing results in tabbed views.
    *   **Integrated Mode:** Runs the full lexer-parser pipeline.
    *   **Standalone Modes:** Generates and tests the lexer or parser independently.
    *   **Full Test Modes:** Executes a suite of predefined test cases for both lexical and syntactic components.

## Prerequisites

*   Python 3.x
*   Required Python libraries:
    *   `customtkinter`
    *   `Pillow`
    *   `graphviz`
*   **Graphviz Software:** The Graphviz command-line tool (`dot`) must be installed and accessible in the system's PATH for DFA graph rendering. Download from [graphviz.org/download/](https://graphviz.org/download/).

Install Python dependencies using pip:

```
pip install customtkinter Pillow graphviz
```

## Project Structure

The project is organized into a `core` backend and a `front` GUI.

```
├── core/
│   ├── automata.py             # NFA, DFA, and related algorithms (Thompson, subset, minimization)
│   ├── lexer_core.py           # Lexer, symbol table, RE file parsing
│   ├── regex_utils.py          # RE preprocessing, infix-to-postfix conversion
│   ├── syntax_tree_direct_dfa.py # Direct DFA construction (syntax tree, followpos)
│   ├── syntactic/
│   │   ├── grammar.py          # Grammar representation and parsing
│   │   ├── slr_generator.py    # SLR table generation (First, Follow, Canonical Collection)
│   │   └── slr_parser.py       # SLR table-driven parser
│   └── config.py               # Configuration constants (e.g., EPSILON)
│
├── front/
│   ├── app.py                  # Main GUI application class and state management
│   ├── callbacks.py            # Functions executed by GUI events (button clicks)
│   ├── controls.py             # Reusable GUI control panels
│   ├── frames.py               # Layout definitions for each application screen/mode
│   ├── ui_formatters.py        # Functions to format data structures for display
│   └── ui_utils.py             # Low-level GUI utility functions
│
├── tests.py                    # Predefined lexical test cases
├── syntactic_tests.py          # Predefined syntactic test cases
├── main.py                     # Main application entry point
└── README.md
```

## Usage

Install prerequisites as listed above, then run the application:

```
python main.py
```

## Interface Overview

*   **Start Screen:** Choose an operation mode:
    *   Integrated Mode
    *   Lexer Generator (Thompson/Followpos)
    *   Full Test Mode (Lexical/Syntactic)

*   **Integrated Mode:**
    *   Input regular expression definitions.
    *   Input a context-free grammar.
    *   Input the source code.
    *   Execute Part 1 (Lexical) to generate tokens.
    *   Execute Part 2 (Syntactic) to parse the generated tokens.
    *   Results for all intermediate steps are displayed in tabbed views.

*   **Lexer Generator Mode:**
    *   Input regular expression definitions and source code.
    *   Use step-by-step controls to generate NFAs, determinize to a DFA, minimize, and tokenize.

*   **Full Test Mode:**
    *   Select a predefined test case from a list.
    *   The system executes all generation and analysis steps.
    *   Review results in the tabbed views.

---

## Input Formats and Syntax

### 1. Lexical Analyzer: Regular Expression Definitions

Define token patterns in the input text area, one per line.

**Format:** `TOKEN_NAME: RegularExpression`

#### Supported Regular Expression Syntax

| Feature             | Syntax        | Description                                       | Example                               |
| ------------------- | ------------- | ------------------------------------------------- | ------------------------------------- |
| Concatenation       | `ab`          | Matches 'a' followed by 'b'.                      | `ab`                                  |
| Alternation (Union) | `a\|b`        | Matches 'a' or 'b'.                               | `if\|else`                            |
| Kleene Star         | `a*`          | Matches zero or more occurrences of 'a'.          | `a*`                                  |
| Positive Closure    | `a+`          | Matches one or more occurrences of 'a'.           | `[0-9]+`                              |
| Optional            | `a?`          | Matches zero or one occurrence of 'a'.            | `https?`                              |
| Grouping            | `(ab)`        | Groups a sequence to apply an operator to it.     | `(ab)+`                               |
| Character Class     | `[abc]`       | Matches any single character from the set.        | `[aeiou]`                             |
| Character Range     | `[a-z]`       | Matches any single character in the range.        | `[a-zA-Z0-9_]`                        |
| Escaped Character   | `\+`          | Matches a literal character that is a meta-char.  | `LPAREN: \(`                          |

#### Special Directives and Behavior

*   **Ignoring Patterns:** To match a pattern but not generate a token (e.g., for whitespace or comments or anything else), append `%ignore` to the definition.

```
# To ignore spaces
WS: [ ]+ %ignore

# To ignore single-line comments
COMMENT: //.* %ignore
```

*   **Reserved Words:** The system automatically identifies reserved words based on a naming convention: if a `TOKEN_NAME` is in all uppercase and its `RegularExpression` is the exact lowercase version, it is treated as a reserved word.

```
# Correctly identified as reserved words
IF: if
ELSE: else
WHILE: while
```

*   **Priority and Ambiguity:** The lexer resolves ambiguity using two rules:
    1.  **Longest Match:** It will always match the longest possible string from the input. For input `forloop`, it will match `forloop` instead of `for`.
    2.  **Definition Order:** If two patterns match the same longest string, the one defined *first* in the input file wins. This is critical for patterns that might be prefixes of others or for giving reserved words priority over general identifiers.

**Example: Manually Prioritizing Reserved Words**  
If the automatic convention is not used, you must list reserved words before the general `ID` pattern to ensure they are matched correctly.

```
# Correct Order
IF: if
ID: [a-zA-Z]+

# Incorrect Order (would tokenize 'if' as ID)
# ID: [a-zA-Z]+
# IF: if
```

#### Common Use Case Examples

*   **Identifiers:** A typical identifier starts with a letter or underscore, followed by letters, numbers, or underscores.

```
ID: [a-zA-Z_][a-zA-Z0-9_]*
```

*   **Numbers:** Integers and floating-point numbers. The optional group `(\.[0-9]+)?` handles the decimal part.

```
NUM: [0-9]+(\.[0-9]+)?
```

*   **Operators:** Simple operators may need to be escaped. Multi-character operators are defined by concatenation.

```
PLUS: \+
ASSIGN: =
EQ_COMP: ==
GTE: >=
```

*   **String Literals:** A simple string literal enclosed in double quotes. `[^"]*` matches any sequence of characters that are not a quote.

```
STRING: "[^"]*"
```

### 2. Syntactic Analyzer: Grammar and Token Stream

#### Grammar Definition Format

Define the context-free grammar using production rules.

| Feature         | Syntax                          | Description                                                              |
| --------------- | ------------------------------- | ------------------------------------------------------------------------ |
| **Production**  | `NonTerminal ::= Symbol1 ...`   | The core rule format.                                                    |
| **Start Symbol**| (Implicit)                      | The left-hand side (LHS) of the *first* production rule.                  |
| **Separator**   | `::=`                           | Separates the LHS from the production body.                              |
| **Alternatives**| `\|`                            | Defines multiple bodies for the same non-terminal on a single line.      |
| **Epsilon**     | `&`                             | Represents an empty (epsilon) production.                                |
| **Spacing**     | (Space)                         | Symbols in the production body should be separated by spaces.            |

**Example: A Grammar for a Simple Block-Scoped Language**

```
# The start symbol is 'Program'
Program ::= StmtList

StmtList ::= Stmt StmtList | &

Stmt ::= AssignStmt | IfStmt | Block

Block ::= { StmtList }

AssignStmt ::= id = E ;

IfStmt ::= if ( E ) Stmt

E ::= E + T | T
T ::= id | num
```

This example shows variable assignment, an `if` statement, blocks with `{ }`, and a list of statements. The terminals (`id`, `=`, `if`, `(`, etc.) must correspond to `TOKEN_NAME`s from the lexical definition.

#### Token Stream Format (for standalone syntactic testing)

Provide a sequence of tokens, one per line. This format simulates the output of the lexical analyzer.

**Format:** `TOKEN_TYPE,ATTRIBUTE`

*   `TOKEN_TYPE` must match a terminal symbol in the grammar.
*   `ATTRIBUTE` is the token's value or pointer. It is optional; if omitted, the comma is also optional.
*   The parser automatically appends the end-of-input marker (`$`).

**Example:**  
Given the source code `x = 10;`, the corresponding token stream input for the parser would be:

```
id,0
=
num,10
;
```

## Output Interpretation

The tabbed display provides details for each stage of the analysis:

*   **Lexical Stages:**
    *   **ER → NFA / Tree+Followpos:** Shows individual NFAs (Thompson) or the augmented syntax tree and followpos table (Direct Method).
    *   **NFA Combined / Direct DFA:** Displays the combined NFA and the resulting non-minimized DFA table.
    *   **Minimized DFA (Final):** The final, optimized DFA table used by the lexer.
    *   **Minimized DFA Drawing:** A graphical rendering of the final DFA.
*   **Syntactic Stages:**
    *   **Grammar Details:** A summary of the parsed grammar, including terminals, non-terminals, and productions.
    *   **First & Follow Sets:** The computed First and Follow sets.
    *   **Canonical Collection:** The set of LR(0) items and the `goto` transitions between them.
    *   **SLR Parse Table:** The final `ACTION` and `GOTO` tables.
*   **Final Output:**
    *   **Lexical Analyzer Output (Tokens):** The list of tokens generated from the source code, e.g., `<'x', ID> (Attribute: index 0)`.
    *   **Symbol Table:** Shows static pattern definitions and the dynamic symbol table populated during tokenization.
    *   **Parse Steps:** A trace of the parser's stack, remaining input, and the action taken at each step.

## Images 

### --- Lexical Analyzer ---


<img width="1673" height="882" alt="image" src="https://github.com/user-attachments/assets/9a7cd2ee-6cc5-44e8-a3ed-3a71d329e872" />
<img width="1659" height="875" alt="image" src="https://github.com/user-attachments/assets/ffaa2b3f-4f43-4cb9-96db-0ce71581ff5e" />
<img width="1666" height="881" alt="image" src="https://github.com/user-attachments/assets/f77596b8-be60-4474-bfc4-73dde042f392" />
<img width="1673" height="880" alt="image" src="https://github.com/user-attachments/assets/76a8ec89-49c0-4e95-9301-e4ed0b7d9304" />
<img width="1669" height="875" alt="image" src="https://github.com/user-attachments/assets/6f794b95-80f8-4e89-a234-21bbfc518422" />
<img width="1672" height="879" alt="image" src="https://github.com/user-attachments/assets/52567c46-6a13-495f-a7d3-32082ad3873e" />



### --- Sintatic Analyzer ---


<img width="1673" height="885" alt="image" src="https://github.com/user-attachments/assets/3d4fb10b-5bf8-4cca-974f-583f139220b2" />
<img width="1675" height="886" alt="image" src="https://github.com/user-attachments/assets/7e26183c-05ca-4055-882c-7200185adc51" />
<img width="1667" height="878" alt="image" src="https://github.com/user-attachments/assets/920eb7d8-2c85-4c62-8036-c9281b34d86e" />
<img width="1664" height="878" alt="image" src="https://github.com/user-attachments/assets/99d188aa-8114-4310-8b8d-810cd77f81eb" />
<img width="1668" height="885" alt="image" src="https://github.com/user-attachments/assets/d019ad0d-449f-450a-98cb-fa26933ab972" />
<img width="1667" height="878" alt="image" src="https://github.com/user-attachments/assets/21b94612-3e8a-4188-9e72-47030ac7eb35" />
<img width="1662" height="871" alt="image" src="https://github.com/user-attachments/assets/428b3c07-6ca4-4668-9bf2-ace23a93b6e2" />

