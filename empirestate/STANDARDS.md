# EmpireStack — Standards

Development, implementation, and coding standards.

---

## Database

### No ORMs as the Abstraction Layer
The database integration layer uses **explicit accessor abstractions** for database interaction. The accessor is the contract; the implementation behind it is blackbox. If an ORM is used, it sits behind the accessor—not as the abstraction itself. The application never contracts with the ORM; ORM is an implementation detail inside the blackbox, not an overarching architectural decision. Choose explicit SQL by default; ORM behind the interface is tenable when used cleanly.

### Data Stores: SQL, NoSQL, Event Streams
SQL, NoSQL, and event log streams (e.g. Kafka) are all valid depending on application. Each has domains where it thrives.

| Store type | Role | Example |
|------------|------|---------|
| **Event stream** | Event convergence center, SSOT for durable events. Under OpenErgo: Kafka (or similar) is SSOT; RabbitMQ (or similar) is ephemeral router. | Kafka, RabbitMQ, Azure Service Bus |
| **SQL** | At-rest relational data, projections, transactional integrity, reporting. | Postgres, SQLite |
| **NoSQL** | At-rest document/key-value data. Can also play a role in event management adjacent to streams. | MongoDB, Redis |

**Right tool for the job.** Choose based on requirements, not ideology. **What is not allowed:** Using NoSQL where SQL thrives, or SQL where NoSQL thrives—the industry mistake of substituting one for the other based on hype rather than fit.

---

## Paradigms

### Object Orientation, AOP, and Functional Programming
- **Object orientation** for core implementations.
- **Pure functions** for injected behavior.

The right combination of OOP, AOP, and functional programming is the key. Each has its place; picking one exclusively is a mistake.

---

## Typing and Design

| Standard | Requirement |
|----------|-------------|
| **Strong typing** | All languages must use strong typing (e.g., Mypy for Python, TypeScript for JavaScript). |
| **Interface-first design** | Define contracts before implementation. |

---

## Language Preference

Python backend preferred.
