---
layout: default
title: PDG & CIA Analysis Tool for Java 1.4
---

<p class="back-link"><a href="/">← Back to Home</a></p>

<h1>Program Dependence Graphs for Change Impact Analysis</h1>

<p><em>CPSC 499: Software Analysis | December 23, 2025 | Mann, Nico, Mandeep</em></p>

<h2>Project Overview</h2>

<p>Understanding the consequences of code changes is fundamental to software maintenance. When you modify a single line in a large codebase, how do you know what else might break? A seemingly small change can ripple through the system in unexpected ways due to data and control dependencies. This project tackles that problem head-on by building a tool that automatically identifies which lines of code may be impacted by a change.</p>

<p>Working as a team of three, we developed a Program Dependence Graph (PDG) based change impact analysis tool specifically for Java 1.4 programs. The tool parses source code, constructs a comprehensive dependency graph capturing both control flow and data flow relationships, and then performs reachability analysis to determine the impact set for any given line change.</p>

<blockquote>
<strong>Our Central Question:</strong> "Given a Java 1.4 program and a line number, can we automatically determine which other lines of code may be impacted by a change to that line, based on control and data dependencies?"
</blockquote>

<p>This wasn't just a theoretical exercise. We evaluated our tool against manually analyzed test cases, achieving 95% precision and 91% recall across diverse programs. The PDG-based approach proved highly effective at capturing transitive dependencies, loop-carried effects, and variable reassignments with minimal false positives.</p>

<h2>Technical Architecture</h2>

<h3>Parsing and Program Representation</h3>
<p>We used JavaParser configured for Java 1.4 grammar to generate Abstract Syntax Trees from source code. Each statement was mapped to its corresponding line number in the original source file, allowing us to report impact analysis results directly in terms of source code lines rather than abstract graph nodes.</p>

<p>Our implementation built on existing parsing and control flow dependency code, focusing on core Java constructs: variable declarations, assignments, conditional statements, loops, and method calls. This foundation gave us consistency and let us concentrate on PDG construction and evaluation.</p>

<h3>Control Dependency Analysis</h3>
<p>Control dependencies determine which statements control whether other statements execute. Think of an <code>if</code> statement: everything inside that block depends on the condition being true.</p>

<p>Our algorithm works iteratively, walking through the control flow graph and tracing all branching points. For each node, we collect dependencies from predecessors and add any branching nodes as direct dependencies. The process repeats until the dependency information stabilizes—a fixed-point iteration that handles even complex nested control structures.</p>

<h3>Data Dependency Analysis</h3>
<p>Data dependencies track how variables flow through a program. If statement A assigns to variable <code>x</code> and statement B uses <code>x</code>, then B depends on A. Simple in concept, but tricky in practice with loops, branches, and multiple definitions.</p>

<p>We implemented reaching definitions analysis using the classic GEN/KILL algorithm:</p>
<ul>
<li><strong>GEN set:</strong> New definitions introduced by a statement</li>
<li><strong>KILL set:</strong> Previous definitions of the same variable that get invalidated</li>
<li><strong>Propagation:</strong> Definitions flow forward through the control flow graph until convergence</li>
</ul>

<p>Special handling was required for compound assignments (<code>+=</code>, <code>-=</code>), array accesses, and loop counters. We also had to carefully parse left-hand sides of assignments to correctly extract variable names, especially in declarations like <code>int sum = 0</code> where both type and variable appear together.</p>

<h3>Change Impact Analysis</h3>
<p>Once the PDG is built, determining impact becomes a graph reachability problem. Given a changed line, we traverse the PDG following dependency edges to find all reachable nodes. Those nodes represent the lines that might be affected by the change.</p>

<p>The quality of the PDG directly determines the quality of the impact analysis. Incorrect dependency edges lead to either missed impacts (false negatives) or spurious impacts (false positives). Our evaluation showed that the PDG construction was robust enough to achieve strong results in both dimensions.</p>

<h2>What I Learned</h2>

<h3>Fixed-Point Algorithms Are Subtle</h3>
<p>Both control and data dependency analysis rely on fixed-point iteration—you keep computing until nothing changes. Sounds simple, but getting convergence guarantees and handling edge cases took careful thinking. Straight-line code converges quickly, but loops and deeply nested conditionals require multiple passes. We learned to balance thoroughness with performance by setting reasonable iteration limits.</p>

<h3>Mapping Abstract Representations to Source Code</h3>
<p>One of our biggest challenges was bridging the gap between CFG nodes (which use internal identifiers) and actual source line numbers. The parse tree represents syntax, the CFG represents control flow, and they don't align perfectly. We built a mapping table by extracting token positions and matching statement text, but edge cases like whitespace variations and synthetic nodes required special handling.</p>

<h3>Static Analysis Has Inherent Limits</h3>
<p>We couldn't achieve 100% accuracy, and that's expected. Static analysis fundamentally deals with approximations. Our tool occasionally over-approximated (reported false impacts) at control flow merge points, choosing conservatism over missing real dependencies. Under-approximation (missing impacts) happened primarily in complex control-flow scenarios. Understanding and documenting these trade-offs was as important as the implementation itself.</p>

<h2>Challenges & Solutions</h2>

<h3>Challenge 1: Line Number Mapping</h3>
<p><strong>Problem:</strong> CFG nodes have internal identifiers, not line numbers. Multiple statements can appear on one line. Synthetic nodes like Entry and Exit have no source correspondence.</p>
<p><strong>Solution:</strong> We traversed the parse tree to extract token positions and built a comprehensive mapping table. For ambiguous cases, we matched statement text content. Synthetic nodes were handled as special cases in the analysis logic.</p>

<h3>Challenge 2: Reaching Definitions Convergence</h3>
<p><strong>Problem:</strong> Loops and branches create scenarios where multiple definitions can reach a single program point. The analysis must iterate until stable.</p>
<p><strong>Solution:</strong> Implemented the GEN/KILL fixed-point algorithm with careful tracking of which definitions are generated and killed at each statement. The algorithm converges quickly for straight-line code and within reasonable bounds even for complex control flow.</p>

<h3>Challenge 3: Variable Definition Parsing</h3>
<p><strong>Problem:</strong> Statements like <code>int sum = 0</code> have both type and variable on the left-hand side, breaking simple parsing logic.</p>
<p><strong>Solution:</strong> Explicitly separated left-hand side from right-hand side, then parsed only the LHS to extract variable names. This fix also revealed issues with compound operators that we subsequently addressed.</p>

<h3>Challenge 4: Control Dependency Propagation</h3>
<p><strong>Problem:</strong> Complex expressions and loops need multiple graph traversals to capture all control dependencies, especially backward dependencies.</p>
<p><strong>Solution:</strong> Propagated dependencies through multiple passes, checking for new dependencies each time. When consecutive passes produce identical results, we know the control dependency graph is complete.</p>

<h2>Results & Evaluation</h2>

<p>We evaluated the tool against manually analyzed test programs, treating the manual analysis as ground truth. For each test case, we compared the Actual Impacted Set (AIS) with the tool's Estimated Impacted Set (EIS) using standard precision and recall metrics.</p>

<p><strong>Performance Metrics:</strong></p>
<ul>
<li>95% Average Precision</li>
<li>91% Average Recall</li>
<li>85% Perfect Accuracy</li>
</ul>

<p>The tool excelled at data-flow driven scenarios, accurately capturing transitive dependencies and loop-carried effects. False positives were rare and limited to conservative control-flow merge points. False negatives occurred primarily when propagating impact from control predicates to deeply nested conditional branches.</p>

<blockquote>
The few imperfect cases revealed a clear pattern: under-approximation happened in control-flow-heavy situations, while over-approximation was minimal and conservative. This aligns with the expected trade-offs of static analysis—we prioritized soundness while maintaining strong precision.
</blockquote>

<h2>Takeaways</h2>

<p>This project demonstrated that PDGs provide a solid foundation for automated change impact analysis. The combination of control and data dependency tracking captures the essential relationships that determine how changes propagate through code.</p>

<p>Key principles we validated:</p>
<ul>
<li>Fixed-point algorithms are the right tool for dependency analysis</li>
<li>Careful handling of edge cases separates working tools from broken ones</li>
<li>Mapping between different program representations is non-trivial</li>
<li>Static analysis trade-offs should be explicit and documented</li>
<li>Ground truth evaluation reveals both strengths and limitations</li>
</ul>

<h3>Limitations</h3>
<p>Our work has important constraints. The manual ground truth may contain errors. We tested on small to medium programs; scalability to large codebases remains unexplored. The project was completed in 1-2 months, limiting scope. And we focused on Java 1.4, so modern language features aren't supported.</p>
<p>These limitations don't invalidate the results—they define the boundaries of what we can claim. Future work could address scalability, extend language support, and explore inter-procedural analysis for method calls.</p>

<h3>Technologies & Concepts Used</h3>
<p>JavaParser | Abstract Syntax Trees | Control Flow Graphs | Program Dependence Graphs | Reaching Definitions Analysis | GEN/KILL Algorithm | Fixed-Point Iteration | Java 1.4</p>

<p class="back-link"><a href="/">← Back to Home</a></p>