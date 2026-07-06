# Basics_of_JVM_Tomcat

```text
				+-------------------------------------------+
                |          End User / Browser               |
                +--------------------+----------------------+
                                     |
                                     | HTTP Request
                                     v
+-----------------------------------------------------------------------------------+
|                              APACHE TOMCAT SERVER                                 |
|                                                                                   |
|   +-------------------------------------------------------------------------+     |
|   |                    CONNECTOR (HTTP / AJP)                               |     |
|   |                    listens on port 8080                                 |     |
|   +-----------------------------------+-------------------------------------+     |
|                                       |                                           |
|                                       v                                           |
|   +-------------------------------------------------------------------------+     |
|   |                        THREAD POOL                                      |     |
|   |   Thread-1   Thread-2   Thread-3   ...   Thread-N                       |     |
|   |   assigns a free worker thread (max threads configured)                 |     |
|   +-----------------------------------+-------------------------------------+     |
|                                       |                                           |
|                       (if all threads busy -> request queues)                     |
|                                       |                                           |
|                                       v                                           |
|   +-------------------------------------------------------------------------+     |
|   |                     WAR FILE (deployed)                                 |     |
|   |         login.war   /   ecommerce.war   /   app.war                     |     |
|   +-----------------------------------+-------------------------------------+     |
|                                       |                                           |
|                                       v                                           |
|   +-------------------------------------------------------------------------+     |
|   |                        SERVLETS                                         |     |
|   |     LoginServlet   OrderServlet   ProductServlet   REST APIs            |     |
|   |     handles request, processes logic, builds response                   |     |
|   +-----------------------------------+-------------------------------------+     |
|                                       |                                           |
+---------------------------------------+-------------------------------------------+
                                        |
                                        | Executes Java Code
                                        v
========================================================================================
                         JVM (Java Virtual Machine)
========================================================================================
|                                                                                      |
|   +-----------------------------------+       +-----------------------------------+  |
|   |          HEAP MEMORY              |       |     NON-HEAP MEMORY               |  |
|   |     (active working data)         |<----->|        (Metaspace)                |  |
|   |                                   |       |                                   |  |
|   |  - User Sessions                  |       |  - Class Definitions              |  |
|   |  - Shopping Carts                 |       |  - Application Metadata           |  |
|   |  - Objects & Variables / Caches   |       |  - Code Blueprints                |  |
|   +----------------+------------------+       +-----------------------------------+  |
|                    |                                                                 |
|                    v                                                                 |
|   +-------------------------------------------------------------------------------+  |
|   |                        GARBAGE COLLECTOR (GC)                                 |  |
|   |                                                                               |  |
|   |   Scavenge GC (Minor / Quick Cleanup)                                         |  |
|   |        -> clears short-lived unused objects                                   |  |
|   |                                                                               |  |
|   |   Mark-Sweep GC (Major / Deep Cleanup)                                        |  |
|   |        -> "Stop-the-World" pause, app freezes briefly to reclaim memory       |  |
|   |                                                                               |  |
|   |   Result: Unused objects removed, Heap space reclaimed                        |  |
|   +-------------------------------------------------------------------------------+  |
|                                                                                      |
========================================================================================
                                       |
                                       v
+-------------------------------------------------------------------------------------+
|                          RESPONSE SENT BACK                                         |
|                    via Connector -> Thread -> End User                              |
+-------------------------------------------------------------------------------------+


        Cross-cutting monitoring / management access point (runs alongside all layers):
        +-----------------------------------------------------------------------------+
        |                    JMX (Java Management Extensions)                         |
        |                                                                             |
        |   Exposes MBeans -> live queryable state of JVM & Tomcat, e.g.:             |
        |     - Thread Pool status (busy/idle threads)                                |
        |     - Heap / Non-Heap memory usage                                          |
        |     - GC stats (frequency, pause duration)                                  |
        +-----------------------------------------------------------------------------+
                                       |
                                       v
+-------------------------------------------------------------------------------------+
|                         OPERATING SYSTEM / HOST                                     |
|                         CPU  |  RAM  |  Disk  |  Network                            |
+-------------------------------------------------------------------------------------+
```

### Flow summary:

- Browser sends an HTTP request → hits Tomcat's Connector.
- Thread Pool assigns a free thread (or queues the request if all are busy).
- Thread executes code inside the deployed WAR file.
- The relevant Servlet (e.g. LoginServlet) processes the request.
- All of this runs inside the JVM — using Heap for live data and Non-Heap/Metaspace for class structures.
- Garbage Collector periodically reclaims unused Heap objects (minor Scavenge or major Mark-Sweep).
- Response flows back out through the same thread and connector to the user.
- JMX/MBeans sits alongside every layer as a read-only interface exposing live metrics.
- Underneath everything, the OS/Host supplies the actual CPU, RAM, disk, and network resources.
