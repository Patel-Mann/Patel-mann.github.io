---
layout: default
title: Notflix - CPSC559 Final Report
---

<p class="back-link"><a href="/">← Back to Home</a></p>

<img class="project-hero-image" src="/assets/images/notflix.png" alt="Notflix Screenshot">

<h1>Notflix - A Distributed Streaming Platform</h1>

<p><em>CPSC559 Final Report | Group 11 | Mann Patel, Ali Al Yasseen, Guanhua Huang (Eric), Thi My Tuyen Tran (Esther)</em></p>

<h2>Abstract</h2>

<p>Notflix is a multi-node, fault tolerant video streaming service built in Go (backend and proxy) and React/Vite (web client), backed by a per node SQLite database. It demonstrates the core mechanisms of a distributed systems: HTTP and net/rpc communication, a persisted monotonic term counter for logical synchronization, primary backup replication with synchronous majority quorum commit and asynchronous catch-up, the Bully algorithm for leader election, leader-only writes, heartbeats with a per follower circuit breaker and pre-leadership log sync for fault tolerance.</p>

<h2>Introduction</h2>

<p>A single monolithic backend is simple but introduces a single point of failure and caps write throughput at one machine. This project's motivation is to build, from scratch, a small but realistic distributed backend that behaves like a simplified Netflix: users can register and log in, browse a catalog of short films, stream video, favorite content and record watch history, while the cluster remains available and consistent under failures and restarts.</p>

<p>The bundled dataset is intentionally lightweight: six open-source short films, twelve genres and the usual relational tables. Each node owns its own SQLite file while the replication layer keeps them converging.</p>

<h2>Architecture</h2>

<p>Notflix has three tiers:</p>
<ul>
<li><strong>The React frontend</strong> talks exclusively to the proxy.</li>
<li><strong>The Go proxy</strong> polls each backend's /cluster/info every 3s, tracks the current leader, and forwards requests accordingly.</li>
<li><strong>The backend cluster</strong> consists of five Go processes on ports 8080 - 8084 (HTTP) and 18080 - 18084 (net/rpc), each with its own SQLite file.</li>
</ul>

<!-- [TODO: Add Architecture Image] -->
<img class="project-inline-image" src="/assets/images/Untitled.png" alt="Architecture Diagram">

<p>A write flows as: proxy > leader > createLeaderMiddleWare > metadata handler inside a GORM transaction > replication_log insert in the same transaction > ReplicateWithQuorumTx broadcast to followers via RPC and blocks for majority ACKs, only then does the transaction commit.</p>

<h2>Communication</h2>

<p>Communication is split into two channels:</p>
<ul>
<li><strong>Clients use HTTP</strong> via gorilla/mux. The /api/* subrouter serves reads and writes on users, content, genres, favorites, watch history and search.</li>
<li><strong>Inter-node traffic uses Go's net/rpc</strong> over TCP. Each node registers Election and Replication RPC services on its RPC port. net/rpc was chosen because it provides synchronous request/response semantics that map naturally to Bully messages.</li>
</ul>

<h2>Synchronization</h2>

<p>Notflix does not need physical clocks across nodes, it needs a total order on committed writes. This is provided by a persisted monotonic term counter and in-process locks.</p>

<p>Every node maintains CurrentTerm, bumped on each election start and flushed to disk. On any inbound message, AdoptTermIfHigher advances the term and steps down from any stale leader role before doing anything else. Persisting the term prevents a restarted node from claiming a term the cluster has already moved past.</p>

<h2>Replication</h2>

<p>Notflix uses primary backup replication with a synchronous majority-quorum commit plus an asynchronous catch-up path.</p>
<ul>
<li><strong>On the leader:</strong> ReplicateWithQuorumTx runs inside the same GORM transaction as the user visible mutation, inserting a replication_log row and broadcasting to all followers in parallel, waiting for majority ACKs.</li>
<li><strong>On a follower:</strong> HandleReplication checks LeaderTerm, idempotently ACKs duplicates and applies entries only when LogID = last_applied + 1.</li>
<li><strong>Catch-up workers:</strong> StartCatchupWor/ker ticks every 4s on non-leaders to pull missing entries.</li>
</ul>

<!-- [TODO: Add Replication Image] -->
<img class="project-inline-image" src="/assets/images/Untitled-3.png" alt="Replication Diagram">

<p>Primary backup with a majority quorum was chosen because the workload is read-heavy, the cluster is small and five nodes tolerating two failures is enough.</p>

<h2>Election</h2>

<p>Leader election uses the classical Bully algorithm, strengthened with a persisted term counter and a pre-leadership log sync hook.</p>

<p>The Bully algorithm was chosen because the cluster is small, statically configured and a deterministic priority-based winner is easy to demonstrate while having the lowest leader recovery time. Priority is the trailing integer of NODE_ID (node-4 beats node-3), so the highest-numbered live node is always the leader.</p>

<p>Key improvements made to fix timing errors:</p>
<ul>
<li>A stable leader that receives an ELECTION from a lower priority node answers but does not adopt the sender's term.</li>
<li>COORDINATOR or HEARTBEAT from a lower priority node is rejected and triggers a fresh election.</li>
<li>The atomic isElecting guard and 2s cooldown prevent re-election storms.</li>
<li>Heartbeats are sent every 2s and the 6s dead-threshold absorbs one dropped packet without false elections.</li>
</ul>

<!-- [TODO: Add Election Image] -->
<img class="project-inline-image" src="/assets/images/Untitled-2.png" alt="Election Diagram">

<h2>Consistency</h2>

<p>Notflix provides strong consistency on writes and close to linearizable reads through the proxy. It combines four rules:</p>
<ul>
<li>Single-leader writes</li>
<li>Redirect-to-leader middleware</li>
<li>Synchronous majority quorum</li>
<li>Ordered replication logs with term-gated follower coordination</li>
</ul>

<!-- [TODO: Add Consistency Image] -->
<img class="project-inline-image" src="/assets/images/Untitled-4.png" alt="Consistency Diagram">

<p>Quorum commit ensures that any acknowledged write is durable on a majority of nodes. Because any future majority must overlap, no newly-elected leader can miss a committed entry.</p>

<h2>Fault Tolerance</h2>

<p>The system is designed to tolerate three specific failure scenarios: a single node crash, a transient network partition and a node restart with stale state.</p>

<h3>Heartbeat Contract</h3>
<p>The leader broadcasts a HEARTBEAT every 2 seconds. If timeSince(leaderNode.LastSeen) exceeds 6 seconds, an election is triggered. The 6-second threshold represents three missed heartbeats, which tolerates a single dropped packet while maintaining responsive failure detection.</p>

<h3>Circuit Breaker</h3>
<p>On a follower, after three consecutive RPC failures, the circuit opens for 15s, during which the peer is skipped on broadcast and excluded from quorum calculations.</p>

<h3>Rejoining Nodes</h3>
<p>StartCatchupWorker runs every 4s, pulling up to 500 missing log entries from the peer with the highest last_applied_log_id. SyncReplicationBeforeLeadership adds a stronger guarantee at promotion time.</p>

<h3>Crash-Safe Storage</h3>
<p>Every application write and its corresponding replication_log insert are committed together in a single atomic transaction, with SQLite's fsync-on-commit ensuring data reaches disk before commit is acknowledged.</p>

<h2>Summary and Conclusions</h2>

<p>Notflix demonstrates that a small, well-factored Go backend is sufficient to cover the core mechanics of a distributed system without heavy infrastructure.</p>

<p>Key takeaways:</p>
<ul>
<li>Splitting communication between HTTP for client-facing traffic and net/rpc for internal cluster messaging keeps each channel well-suited to its purpose.</li>
<li>A stored counter acts like a logical clock that only moves forward, helping prevent split-brain.</li>
<li>The primary-backup setup uses majority approval for commits so fast writes aren't slowed down.</li>
<li>The Bully algorithm, improved with term checks and a sync step before leadership, ensures predictable and reliable failover.</li>
</ul>

<h3>Limitations</h3>
<ul>
<li>All writes funnel through a single leader, which is acceptable for a small platform but insufficient for a real streaming backend at scale.</li>
<li>Cluster membership is static, meaning topology changes require a coordinated restart.</li>
</ul>

<h3>Technologies Used</h3>
<p>Go | net/rpc | gorilla/mux | GORM | SQLite | React | Vite | HTTP | TCP</p>

<p class="back-link"><a href="/">← Back to Home</a></p>
