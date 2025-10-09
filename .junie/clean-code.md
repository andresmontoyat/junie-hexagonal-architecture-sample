# Clean Code for Hexagonal Kotlin Services

Goals
- Keep business logic independent from frameworks and transport/persistence details.
- Small, cohesive units with clear responsibilities. Prefer composition over inheritance.

Principles
- Single Responsibility: each class has one reason to change.
- Open/Closed: extend via new classes/adapters; do not modify domain for infrastructure needs.
- Dependency Inversion: domain and application declare ports; infrastructure implements them.
- Tell, Don’t Ask: encapsulate behavior inside domain objects/use cases.
- YAGNI/KISS: do the simplest thing that works; avoid speculative generalization.
- Immutability: prefer immutable domain models and DTOs.

Layers in this repo
- Domain (pure): entities/value objects, enums, domain services if any. No Spring, no I/O.
- Application (use cases): orchestrate domain logic, define inbound/outbound ports, enforce application rules. No persistence/messaging/web.
- Infrastructure (adapters): implementations of outbound ports (DB, Kafka), and entry points (REST/GraphQL/Kafka consumer). Contains frameworks and I/O.
- Bootstrap: application wiring and configuration to assemble the runtime.

Use cases
- Keep as thin as possible but the single place for orchestration and transactions.
- Depend only on ports defined in application.
- Validate inputs and enforce application rules; domain invariants live in domain.

Ports
- Inbound ports: interfaces representing use case API consumed by entry points.
- Outbound ports: interfaces representing dependencies needed by use cases.
- Suffixes: `UseCasePort` for inbound; `Repository`, `Publisher`, etc., for outbound.

Adapters
- Entry points (REST/GraphQL/Kafka): map transport to inbound ports, handle validation and response mapping, no business logic.
- Driven adapters (JDBC/Kafka producer): implement outbound ports, map domain to technology-specific types, handle technical errors.

Mapping
- Create dedicated mappers for each boundary (REST, GraphQL, JDBC, Kafka). Keep pure functions.
- Do not place mapping logic inside use cases or domain objects.

Transactions
- Start and commit/rollback around use case methods that change state. In Spring, annotate the use case implementation with `@Transactional` (application layer) rather than repositories.
- Use lambas for try to control the transactions
````
fun interface Transaction<R> {
    fun run(callback: TransactionCallback<R>): R
}

fun interface TransactionCallback<R> {
    fun call(): R
}
````

Errors
- Prefer meaningful exceptions or sealed error results in domain/application.
- Centralize exception-to-HTTP/GraphQL error mapping in handlers.

Testing strategy
- Domain: pure unit tests without mocks.
- Use cases: unit tests mocking outbound ports; verify behavior and interactions.
- Adapters: slice/integration tests (e.g., MyBatis with Testcontainers), and contract tests for REST/GraphQL.
- End-to-end (optional): thin happy path flows using docker-compose/Testcontainers.

Code smells to avoid
- Anemic domain where all logic is in controllers or repositories.
- Leaking Spring or persistence annotations into domain/application modules.
- God services with many responsibilities; break down by use case.
- Overuse of static utility; prefer domain services or extension functions scoped appropriately.

Tips
1. Avoid NULLs
2. Fakes > mocks
3. Keep It Simple
4. Use solid IDEs
5. Names > comments
6. Divide & conquer
7. Write fast tests
8. Use strong names
9. Subtypes must fit
10. Minimize comments
11. Delete unused code
12. Keep cohesion high
13. Test early & often
14. Master IDE hotkeys
15. Set max line width
16. Remove noise words
17. Avoid magic numbers
18. Avoid magic strings
19. Use auto-formatters
20. Avoid large classes
21. Commit early & often
22. Working ≠ clean code
23. Comments explain why
24. Prefix your booleans
25. Use searchable names
26. Don't Repeat Yourself
27. Avoid long conditions
28. Write small functions
29. Use consistent naming
30. No extensive comments
31. Link commits to tasks
32. Keep interfaces small
33. Avoid global variables
34. Capture business logic
35. Write repeatable tests
36. Refactor early & often
37. Produce thorough tests
38. Remove not needed code
39. Depend on abstractions
40. Use pronounceable names
41. Keep proper indentation
42. Write independent tests
43. Don't use abbreviations
44. Max 8-10 lines/function
45. Use parameterized tests
46. Strive for low coupling
47. No horizontal alignment
48. Use AAA pattern in tests
49. Readability > cleverness
50. Limit function arguments
51. Use meaningful test data
52. Readability > efficiency
53. Don't use boolean params
54. Hard-to-test = bad smell
55. Use formatting standards
56. Don't use logic in tests
57. One responsibility/class
58. Write meaningful commits
59. Write deterministic tests
60. Hide irrelevant test data
61. Use feature-based folders
62. Use comments for API docs
63. Use nouns for class names
64. Do real-time code reviews
65. Use consistent vocabulary
66. Avoid primitive obsession
67. Composition > inheritance
68. Avoid long parameter list
69. Use Should/When test names
70. Have one behavior per test
71. Use descriptive test names
72. Write self-validating tests
73. Don't use negative booleans
74. Use adjectives for booleans
75. Write clean test assertions
76. One public method per class
77. Pair-programming on default
78. Don't use encodings in names
79. Use present tense in commits
80. Leave code cleaner you found
81. Reduce cyclomatic complexity
82. One responsibility per module
83. Write code for humans to read
84. Use verbs for functions names
85. Use intention-revealing names
86. One responsibility per module
87. Use imperative mode in commits
88. Use enums as flags in functions
89. Don't state obvious in comments
90. Bundle data & functions together
91. Declare variables close to usage
92. Use empty lines to separate logic
93. Order functions by execution order
94. Strive for small private functions
95. Write code that reads like a prose
96. Use comments for implicit behaviours
97. Use 3-second rule for function names
98. Tests should be as clean as prod code
99. Use rule of three to remove duplication
100. Strive for no side effects in functions