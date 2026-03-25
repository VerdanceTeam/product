# Skill Catalog

Browse available PM skills below. Each skill is a portable AI prompt template that works with any LLM.

To create a new skill, see the [template](_template.md).

> **Note:** These categories are a first cut and will be revisited as skills are added and our PM practice evolves. Expect categories to be renamed, merged, or restructured over time.

## Skills by Category

### Discovery
| Skill | Description | Usage |
|---|---|---|
| _No skills yet_ | [Create one](../skills/_template.md) | — |

### Strategy
| Skill | Description | Usage |
|---|---|---|
| _No skills yet_ | [Create one](../skills/_template.md) | — |

### Product Definition
| Skill | Description | Usage |
|---|---|---|
| _No skills yet_ | [Create one](../skills/_template.md) | — |

### Tactical Execution
| Skill | Description | Usage |
|---|---|---|
| _No skills yet_ | [Create one](../skills/_template.md) | — |

### Service Design
| Skill | Description | Usage |
|---|---|---|
| _No skills yet_ | [Create one](../skills/_template.md) | — |

### Stakeholder Management
| Skill | Description | Usage |
|---|---|---|
| _No skills yet_ | [Create one](../skills/_template.md) | — |

### Measurement
| Skill | Description | Usage |
|---|---|---|
| _No skills yet_ | [Create one](../skills/_template.md) | — |

### Hiring
| Skill | Description | Usage |
|---|---|---|
| _No skills yet_ | [Create one](../skills/_template.md) | — |

### Meta
| Skill | Description | Usage |
|---|---|---|
| [eureka](meta/eureka.md) | Capture an in-the-moment lesson and route it to a skill or GitHub issue | `/eureka [lesson]` |
| [elementary](meta/elementary.md) | Scan context or the current conversation for lessons, write them to a reflections file, then promote selected ones to Eureka moments | `/elementary [context]` |

### Engineering
| Skill | Description | Usage |
|---|---|---|
| [reverse-engineer-architecture](engineering/reverse-engineer-architecture.md) | Generate architecture docs from a codebase by reading code first — with guardrails against 10 AI documentation failure modes | `/reverse-engineer-architecture [scope]` |
| [validate-architecture](engineering/validate-architecture.md) | Validate existing architecture docs against source code using parallel ground truth extraction, hook attribution greps, diagram verification, and negative existence checks | `/validate-architecture [focus]` |
| [reverse-engineer-specifications](engineering/reverse-engineer-specifications.md) | Generate Gherkin behavioral specs from a codebase by tracing routes, role checks, status flows, and data files — organized around the actual navigation structure | `/reverse-engineer-specifications [scope]` |

### Uncategorized
| Skill | Description | Usage |
|---|---|---|
| _No skills yet_ | [Create one](../skills/_template.md) | — |

## How to Add a Skill to This Catalog

When you create a new skill, add a row to the appropriate category table above:

```markdown
| [skill-name](category/skill-name.md) | Brief description of what it does | `/skill-name your input` |
```
