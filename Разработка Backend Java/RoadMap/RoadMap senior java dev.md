```
General IT knowledge
    Application maintenance
        Logging
            Async vs sync logging
            What information must be included into logging message
            ELK
        Application Monitoring
            Health check probes
        Crash dump analysis
        Tracing based on service-mesh
        Application Design Patterns
            Monolithic architecture
                Pros and cons of monolithic architecture vs mircoservices
            Event-driven Architecture
            Microservices Architecture
                Service mesh
                Service registry
                Transactional outbox
                Remote procedure invocation
                Messaging pattern
                Circuit Breaker
                Saga pattern
    How OSs work in General
        How random access memory works
        Understanding process concept
        Understanding thread concept
        Understand stack memory concept
        File System
        Linux basic commands
            Linux command: grep
            Linux command: ls
            Linux command: tail
            Linux command: cat
            Linux command: chmod
    Working with data
        Data Formats
            Fixed-length format
            Protocol Buffers
            JSON
            Yaml
            Binary Serialization
            Xml
            CSV
        Web Services
            Authentication
                OpenID
                SAML
                JWT
                OAuth 2.0
            Internet Protocols
                REST
                SOAP
                FTP
        Databases
            Relational Databases
                PostgreSQL
                SQL migrations
                MySQL
                Oracle
                SQL commands: INSERT, UPDATE, DELETE, SELECT
                Using Inner Joins
                Using Left Joins
                Sequence
                Stored Procedure
                Understanding SQL plan
                Single-Column Index
                    Multi-column Index
                    Understanding index direction
                    Primary Index
                    Cluster index
                Database partitioning
                Auto-increment values
                SQL Transactions
                    Transaction Isolation levels
                Aggregate Functions
            Database Normalization
            Database replication
            Database Sharding
            NoSQL Databases
            Column-oriented databases
    Math for Programmers
        Set Theory
        Category theory
        Algebra
            Predicate
            Commutativity
            Distributivity
            Associativity
        Mathematical Logic
        Geometry
            Cartesian Coordinate System
    Application lifecycle
        Version Control Systems
            Git
                Git commit
                    Git tagging
                    Git stash
                    Git stage
                Gitflow
                Git Branching
                    Git merge
                        Git rebase
                        Git squash
                Remote vs local repository
                    Git fetch
                    Git push
                    Git pull
                .gitignore
                Git limitations
        Java build systems
            Maven
            Gradle
            Package dependency
                Dependency scope
            Package managers
                Nexus
            Java application versioning
        Containers
            Docker
                Docker Installation and Setup
                Docker Images
                Docker Containers
                Container Networking
                Docker volumes
                Docker compose
                Docker registry
                Docker container orchestration
                Docker Security
                Docker Monitoring and Logging
            Kubernetes
                Cluster Setup and Configuration
                Containerization
                Pod Management
                Deployment
                Scaling
                Service and networking
                Persistent Storage
                Monitoring and Logging
                Security and Access Control
                Troubleshooting and Debugging
                Cluster Maintenance and Upgrades
            OpenShift
    Code Quality
        SOLID
        Code testing
            Unit Testing
            Functional Testing
            Integration Testing
            Performance Testing
        DRY
        Application maintainability principles
        KISS
    Algorithms
        Binary Search
        Depth-first search algorithm
        Breadth-first algorithm
        Comparison metrics
            Time complexity notations
            Worst time complexity
            Average time complexity
            Memory consumption of algorithm
            Stable sorting
        Quicksort
        Heapsort
        Counting sort
Core Java
    Java syntax
        Keywords
        Control Flow
        Arrays
        Packages and Imports
        Java I/O
        Strings and Regular Expressions
        Object-oriented programming
            Interfaces vs abstract classes
        Exceptions
            Runtime handing of exceptions
            Fatal errors
        Identifiers
        Literals
        Operators
        Types
            Generic types
                Type erasure
                Wildcards in generics
                Heap pollution
            Primitive types
                Boxing and unboxing
                Math number vs programming numbers
                    Number overflow
                    Number precision
                    Decimal vs double
                    Errors when working with floating numbers
                    Rounding of floating numbers
                        Fair rounding problem
                    Primitive type memory size
            Reference types
                Anonymous classes
            Raw types
            Casting conversion
                Widening primitive conversion
                Widening and narrowing primitive conve
                Narrowing primitive conversion
                Boxing conversion
                Unboxing conversion
                Widening Reference Conversion
                Narrowing Reference Conversion
        Variables
            final variables
                Understanding the restrictions on using final variables in lambda expressions
                Capturing final variables in lambda expressions
                Optimizations achieved through final variables by the Java compiler
                Using final variables in performance-critical code sections
                Thread safety of final variables
            Variable Naming Conventions
            Variable Memory and Stack/Heap Allocation
                Understanding the memory allocation for variables in the stack and heap
                Knowing the difference between stack-based and heap-based variables
                Managing memory usage and avoiding memory leaks
    Java collections
        List
            ArrayList
            LinkedList
        Queue
            BlockingQueue
            PriorityQueue
            ArrayDeque
        Set
            EnumSet
            HashSet
        Map
            HashMap
            WeakHashMap
            ConcurrentHashMap
            TreeMap
        Array
        Common pitfalls with collections
            How to evade ConcurrentModificationException
            Correct Equals and HashCode implementation
            Adding mutable object to hash-based collection
        Streams
            Performance of streams vs collections
            Lazy evaluation in Streams
    Concurrency
        Understanding what is Process
        Understanding what is Thread
            What happens when thread sleeps
                What is thread interrup
                    Interrupt status flag
            What is thread stack
            ThreadLocal variables
        Memory Model
            Happens-before principle
            Atomicity
            Race condition
            Update visibility
            Operations re-ordering
            volatile keyword
            Immutable objects
        Synchronization Patterns
            Understanding the monitor locking
            Synchronized keyword
            Lock
                ReadWriteLock
                ReentrantLock
            Condition interface
            Reentrancy
        Common multhi-thread problems
            Deadlock
            Livelock
            Starvation
        Executors
            ThreadPool
            ForkJoinPool
Java Frameworks and Libraries
    Spring Boot
        Spring Data JPA
        Application properties
            Application profile
        Bean
            Bean lifecycle
            InitializingBean
        Common spring annotations
            @Prototype
            @Configuration
            @ConfigurationProperties
            @Autowire
            @Service
            @Component
            @Transaction
    Apache Commons
    Log4j
    Mockito
    JUnit
    Apache Tomcat
    Java ORMs
        Hibernate
        MyBatis
        Jooq
How JVM works
    Java memory organization
        Heap memory
            Memory Analysis and Profiling
            Memory optimization technics
                Memory optimization: object reuse
                Memory optimization: minimize object creation
                Memory optimization: usage of local variables
                Memory optimization: do not concatenate strings in loops
                Memory optimization: weak references
                Memory optimization: proper resource disposal
            JVM flags to control heap memory sizes
            Structure of JVM heap memory
                Eden Space
                Survivor Spaces
                Tenured Generation
                Permanent Generation
        Method Area
        Stack memory
            Understanding Stack pointer
            Understanding Stack Frame
            Understanding Stack size
            Usage of stack during method invocation
            Usage of stack and recursive calls
                Tail chain optimization
        Garbage collection
            Object reference
                WeakReference
            Memory leak detection
            Object finalization
            Tuning garbage collection
            GC root
            Mark and sweep algorithm
    Class loaders
        Class Loader Hierachy
        Custom Class Loading
        Class Loading Policies
        Class Loader Isolation
        Dynamic Class Loading
    Execution engine
        JIT compiler
            Intermediate Code Generator
            Code Optimizer
                Dead code elimination
                Constant folding and propagation
                Method inlining
                Escape analysis
            Target Code Generator
            JIT profiler
        Interpreter
    Native Method Interface
    Common JVM errors
        ClassNotFoundExcecption
        NoClassDefFoundError
        OutOfMemoryError
        StackOverflowError
How the Internet works
    Transport protocols
        TCP / IP model
            IP Address
            Network port
                Local port
                Remote port
            MAC Address
            Network route
        Proxy mechanism
            Reverse proxy
            Forward proxy
        DNS
            DNS resolve mechanism
            DNS alias
    Basics of HTML
    Browsers
        Browser Caching
        URL
        Request compression
        Understanding the Cookie
    Application protocols
        HTTP protocol
            GET request
            POST request
            PUT request
            HEAD request
            DELETE request
            HTTP header
                Application content header
                Cookie
            Timeout limitation
            Resource usage
            Athentication mechanisms
                Basic Authentication
                Token Authentication
            Long polling
            HTTPS protocol
                SSL/TLS
                Digital signature
                Certificate chain
                mTLS
            SSE
        GraphQL
        WebSockets
        HTTP/2 protocol
    Internet Security
        Same-origin security policy
        Cross-origin resource sharing (CORS)
        Web Application Security Threats
            Arbitrary file inclusion
            Cross-site request forgery
            Cross-site scripting
            Path disclosure
            SQL Injection
            Denial-of-service atack
Teamwork
    Collaboration
    Conflict resolution
    Persuading
    Coordinating
    Creative Thinking
    Active listening
    Responsibility
    Communication skills
    Goal Setting
    Honesty
    Give Feedback
    Empathy
User Interface Disign Patterns
    Model-View-View-Model
    Model-View-Presenter
    Model-View-Controller
```