# AL Development Configuration

This memory file contains AL (Application Language) development guidelines and patterns for Microsoft Dynamics 365 Business Central.

## BC Code Intelligence MCP

This profile includes the BC Code Intelligence MCP server, which provides access to 14 specialized Business Central domain experts and structured development workflows.

### Available Specialists
- **Development:** Dean Debug, Quinn Quality, Isaac Integration
- **Architecture:** Alex Architect, Uri UX
- **Performance:** Pat Performance
- **Security:** Sam Security
- **Testing:** Terra Test
- **Documentation:** Dana Docs
- **Learning:** Leo Learning
- And more covering various BC development domains

### Key Tools
- `ask_bc_expert` - Ask questions to domain specialists
- `discover_specialists` - Get intelligent routing suggestions
- `suggest_specialist` - Start specialist conversations
- `handoff_to_specialist` - Transfer context between specialists
- `start_bc_workflow` - Initiate structured workflows (optimization, review, security audit, etc.)
- `find_bc_topics` - Search for topics by expertise area

### When to Use BC Code Intelligence
- **Complex problems:** Route questions to appropriate domain experts
- **Code reviews:** Use architecture or quality specialists
- **Security concerns:** Consult security specialist
- **Performance issues:** Engage performance expert
- **Best practices:** Get guidance from relevant domain specialists
- **Workflows:** Use structured workflows for optimization, upgrades, testing, onboarding, etc.

## AL Language Conventions

### Object Naming
- Use PascalCase for all objects (tables, pages, codeunits, reports)
- Prefix custom objects with company/app abbreviation
- Table names should be singular nouns (e.g., `Customer`, `SalesHeader`)
- Page names should match table names with type suffix when needed (e.g., `CustomerCard`, `CustomerList`)
- Codeunit names should be descriptive of their purpose

### Field Naming
- Use PascalCase for field names
- Boolean fields should start with verbs (e.g., `IsBlocked`, `HasCustomer`)
- Date fields should end with `Date` (e.g., `PostingDate`, `ShipmentDate`)
- Avoid abbreviations unless industry standard

### Code Style
- Always use explicit typing, avoid `variant` when possible
- Prefer local procedures over global when appropriate
- Use meaningful variable names (e.g., `Customer` not `Cust`)
- Add XML documentation comments for public procedures
- Group variables logically (parameters, return values, local vars)

### AL Best Practices
- Use table extensions instead of modifying base tables
- Implement proper error handling with meaningful error messages
- Prefer events (subscribers) over modifying base application code
- Use temporary tables for processing when appropriate
- Always validate user input in pages and APIs
- Follow the single responsibility principle for codeunits

### Testing
- Write test codeunits for critical business logic
- Use test isolation principles (setup, execute, verify, teardown)
- Mock external dependencies
- Test both positive and negative scenarios

### Performance Considerations
- Use SetLoadFields to optimize data loading
- Avoid unnecessary loops, prefer FindSet for iteration
- Use filtering before loading records
- Be cautious with LOCKTABLE in multi-user scenarios
- Consider batch operations for bulk data processing

## Common AL Patterns

### Event Pattern
```al
[EventSubscriber(ObjectType::Table, Database::Customer, 'OnBeforeInsertEvent', '', false, false)]
local procedure OnBeforeInsertCustomer(var Rec: Record Customer)
begin
    // Custom logic here
end;
```

### Error Handling Pattern
```al
if not SomeCondition then
    Error('Clear error message: %1', Value);
```

## Project Structure
- Keep related functionality together in subfolders
- Separate tables, pages, codeunits, reports into dedicated folders
- Use consistent file naming: `ObjectType.ObjectName.al`

## Commands to Remember
When working with AL projects:
- `alc` or `ctrl+shift+b` - Compile current project
- Use AL Language extension features for navigation
- Leverage code snippets (t, p, c, r for table, page, codeunit, report)

---

*Note: This configuration is shared across your AL development projects. Update it when you discover new patterns or best practices, and all your projects will benefit.*
