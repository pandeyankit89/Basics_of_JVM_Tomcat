## Basics of JVM & Apache Tomcat

```text
                +-------------------------------------------+
                |          End User / Browser               |
                +--------------------+----------------------+
                                     |
                                     | HTTP Request
                                     v
+--------------------------------------------------------------------------------------+
|                              APACHE TOMCAT SERVER                                    |
|                                                                                      |
|  +================================================================================+  |
|  ||                   CATALINA (Servlet Container Engine)                        ||  |
|  ||     manages lifecycle of Connector, Threads, and deployed web app contexts   ||  |
|  ||                                                                              ||  |
|  ||  +-------------------------------------------------------------------------+ ||  |
|  ||  |                "COYOTE" CONNECTOR (HTTP / AJP)                          | ||  |
|  ||  |                listens on port 8080                                     | ||  |
|  ||  +-----------------------------------+-------------------------------------+ ||  |
|  ||                                      |                                       ||  |
|  ||                                      v                                       ||  |
|  ||  +-------------------------------------------------------------------------+ ||  |
|  ||  |                        THREAD POOL                                      | ||  |
|  ||  |   Thread-1   Thread-2   Thread-3   ...   Thread-N                       | ||  |
|  ||  |   assigns a free worker thread (max threads configured)                 | ||  |
|  ||  +-----------------------------------+-------------------------------------+ ||  |
|  ||                                      |                                       ||  |
|  ||                     (if all threads busy -> request queues)                  ||  |
|  ||                                      |                                       ||  |
|  ||                                      v                                       ||  |
|  ||  +-------------------------------------------------------------------------+ ||  |
|  ||  |                     WAR FILE (deployed)                                 | ||  |
|  ||  |         login.war   /   ecommerce.war   /   app.war                     | ||  |
|  ||  +-----------------------------------+-------------------------------------+ ||  |
|  ||                                      |                                       ||  |
|  ||                     +----------------+-----------------+                     ||  |
|  ||                     |                                  |                     ||  |
|  ||                     v                                  v                     ||  |
|  ||  +------------------------------+     +---------------------------------+    ||  |
|  ||  |    JASPER (JSP Engine)       |     |          SERVLETS               |    ||  |
|  ||  |  compiles .jsp files into    |---->|  LoginServlet, OrderServlet,    |    ||  |
|  ||  |  Java source -> .class       |     |  ProductServlet, REST APIs      |    ||  |
|  ||  |  (JSP becomes a Servlet      |     |  handles request, processes     |    ||  |
|  ||  |   at runtime, then runs      |     |  logic, builds response         |    ||  |
|  ||  |   like any other Servlet)    |     |                                 |    ||  |
|  ||  +------------------------------+     +---------------------------------+    ||  |
|  ||                                                    |                         ||  |
|  +=====================================================|==========================+  |
|                                                        |                             |
+--------------------------------------------------------+-----------------------------+
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
        |     - Catalina/Thread Pool status (busy/idle threads)                       |
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
- Catalina is drawn as an outer wrapper around Connector, Thread Pool, and WAR/Servlet handling — because Catalina is the servlet container (tomacat engine); everything from request-routing to app lifecycle happens under its management.
- Jasper sits parallel to the Servlets box, fed by the deployed WAR — it only activates when the request maps to a .jsp file (e.g. login.jsp, index.jsp from your earlier WAR example). It compiles that JSP into a .class file, which then behaves exactly like LoginServlet from that point forward — hence the arrow feeding into the Servlets box.
- Both still ultimately hand off execution into the JVM the same way plain Servlets do.

  ---
```text
+----------------------------------+-----------------------------------------+
|               JAR                |                  WAR                    |
+----------------------------------+-----------------------------------------+
| Java library/application         | Web application                         |
+----------------------------------+-----------------------------------------+
| Contains classes and resources   | Contains Servlets, JSPs, HTML, CSS, JARs|
+----------------------------------+-----------------------------------------+
| Extension = .jar                 | Extension = .war                        |
+----------------------------------+-----------------------------------------+
| Can run standalone (sometimes)   | Deployed in Tomcat                      |
+----------------------------------+-----------------------------------------+
```
- Example structure of a WAR file:
```text
ecommerce.war
│
├── index.jsp
├── login.jsp
└── WEB-INF
    ├── classes
    └── lib
        ├── mysql.jar
        ├── spring.jar
        └── log4j.jar
```
---
