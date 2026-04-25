---
layout: default
title: PDG & CIA Analysis Tool for Java 1.4
---

<p class="back-link"><a href="/">← Back to Home</a></p>

<img class="project-hero-image" src="/assets/images/pdg-cia-tool.gif" alt="PDG & CIA Tool Screenshot">

<h1>Program Dependence Graphs for Change Impact Analysis</h1>

<p><em>CPSC 499: Software Analysis | December 23, 2025 | Mann, Nico, Mandeep</em></p>

<h2> Introduction </h2>

<p>Understanding how a small change affects a large codebase is one of the hardest parts of software maintenance. A single modified line can quietly influence many others through hidden dependencies, whether through control flow or data flow. For this project, we set out to answer a straightforward but important question: given a Java 1.4 program and a specific line, can we automatically determine which other lines might be impacted by a change? To explore this, we built a Program Dependence Graph (PDG)-based analysis tool that models both control and data dependencies and uses that structure to trace how changes propagate through code. The goal was not just theoretical correctness, but something practical and useful for developers trying to understand risk before making edits.</p>

<h2> System Design </h2>

<p>The system works by first parsing Java source code using JavaParser configured for Java 1.4, generating an Abstract Syntax Tree (AST) that represents the structure of the program. From there, we construct a control flow graph and layer on dependency information to build the PDG. Control dependencies capture which statements determine whether others execute, such as conditions in if-statements or loops, and are computed iteratively until the analysis stabilizes. Data dependencies track how variables are defined and used across the program, which we implemented using reaching definitions analysis with the GEN/KILL algorithm. This allows us to follow how values propagate even through loops and branching paths. Once the PDG is built, impact analysis becomes a graph traversal problem: starting from a changed line, we follow dependency edges to find all reachable statements, which form the estimated impact set.</p>

<h2> Challenges and Insights </h2>

<p>While the idea sounds clean in theory, several parts of the implementation turned out to be more subtle than expected. Fixed-point iteration, which underpins both control and data dependency analysis, requires careful handling to ensure convergence, especially in the presence of loops and nested conditionals. Another challenge was mapping abstract graph nodes back to actual source code lines, since internal representations like CFG nodes do not align neatly with the original file. We addressed this by extracting token positions from the AST and building a mapping layer, though edge cases such as multiple statements per line and synthetic nodes required extra handling. Parsing variable definitions also introduced complications, particularly when dealing with declarations that include both types and assignments. Separating and correctly interpreting the left-hand side of statements was necessary to ensure accurate dependency tracking. Throughout the process, it became clear that static analysis inherently involves trade-offs, and that choosing when to over-approximate or risk missing dependencies is a key design decision.</p>

<h2> Results and Takeaways </h2>

<p>In the end, our evaluation showed that the approach was effective. Across a set of manually analyzed test programs, the tool achieved 95% precision and 91% recall, indicating that it captured most real dependencies while keeping false positives low. It performed especially well in scenarios driven by data flow, such as transitive dependencies and loop-carried effects. The remaining inaccuracies largely stemmed from complex control-flow interactions, which is consistent with known limitations of static analysis. Beyond the numbers, the project reinforced several important ideas: Program Dependence Graphs provide a strong foundation for reasoning about change impact, fixed-point algorithms are essential for capturing dependencies accurately, and careful handling of edge cases is what separates a functional tool from a fragile one. While our implementation is limited to Java 1.4 and relatively small programs, it demonstrates that automated change impact analysis is both feasible and valuable, with clear paths for future improvements such as scaling, modern language support, and inter-procedural analysis.</p>