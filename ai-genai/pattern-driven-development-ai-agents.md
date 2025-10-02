# Example-Driven Development: Using Working Code Examples to Guide AI Agents

## The Challenge of AI-Assisted Implementation

When working with AI coding agents, a common challenge emerges: figuring out what actually works can take significant trial and error. Complex integrations, framework-specific configurations, and multi-step workflows often require multiple iterations before achieving a working solution. Once you've successfully implemented a pattern, how do you ensure it can be reliably replicated across projects?

## The Solution: Working Example Repositories

Example-driven development addresses this challenge by capturing proven implementations in example repositories that serve as reference patterns for AI agents. Instead of rediscovering solutions each time, you create working examples that document both the "what" and the "how" of successful implementations.

### What Makes a Good Example Repository?

An example repository is more than just sample code. It combines:

- **Working implementation**: Fully functional code that demonstrates the pattern in action
- **Clear documentation**: Explains the problem being solved and why this approach works
- **Configuration examples**: Shows all necessary setup, dependencies, and environment configuration
- **Reference architecture**: Provides a template structure that can be adapted to other projects

## Real-World Examples

### Kubernetes Remote Debugging with VS Code

The [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug) repository demonstrates a complex multi-step pattern:

**Problem**: Setting up remote debugging for applications running in Kubernetes clusters involves orchestrating multiple tools:
- VS Code Remote Development extension
- Kubernetes port forwarding
- Container runtime configuration
- Debug adapter protocol setup

**Pattern Solution**: The repository provides:
- Complete VS Code configuration files (.devcontainer, launch.json)
- Kubernetes manifests with debugging capabilities enabled
- Step-by-step documentation of the connection flow
- Troubleshooting guide for common issues

**AI Agent Usage**: When implementing remote debugging in a new project, you can reference this repository:
```
I need to set up remote debugging for my Kubernetes application.
Please reference the pattern at https://github.com/nathanfox/k8s-vscode-remote-debug
and adapt it to my project structure.
```

The AI agent can examine the working example and adapt the pattern to your specific:
- Container images
- Kubernetes configuration
- Development workflow
- Framework requirements

### Rapid UI Prototyping System

The [rapid-ui-prototype-example](https://github.com/nathanfox/rapid-ui-prototype-example) repository demonstrates an architectural pattern:

**Problem**: Building a prototyping system that mirrors production components while using mock data requires careful separation of concerns and state management.

**Pattern Solution**: The repository shows:
- Directory structure separating prototypes from production code
- Mock data store implementation patterns
- Scenario-based testing framework
- Component synchronization strategies

**AI Agent Usage**: Reference this pattern to implement a prototyping system in any framework:
```
I want to add a rapid prototyping capability to my React application.
Please analyze the pattern at https://github.com/nathanfox/rapid-ui-prototype-example
and create an equivalent structure for my project, adapting the Vue.js examples to React.
```

## Creating Your Own Example Repositories

### 1. Identify Reusable Examples

Look for implementations that:
- Solve complex, multi-step problems
- Required significant troubleshooting to get working
- Have broad applicability across projects
- Involve specific configurations or integrations

### 2. Extract to a Minimal Example

Create a standalone repository that:
- Removes project-specific dependencies
- Includes only the essential code demonstrating the pattern
- Provides a working example that can be run/tested
- Documents the minimum requirements

### 3. Document the Pattern

Include:
- **Problem statement**: What challenge does this solve?
- **Solution overview**: High-level approach
- **Implementation details**: Step-by-step explanation
- **Configuration examples**: All necessary setup files
- **Usage instructions**: How to run/test the example
- **Adaptation guide**: How to modify for different use cases

### 4. Add AI Agent Instructions

Create a section specifically for AI agents that explains:
- How to analyze the pattern
- What parts are essential vs. optional
- How to adapt to different frameworks/languages
- Common variations and their tradeoffs

## Using Examples with AI Agents

### Effective Prompting Strategies

When referencing an example repository:

```markdown
**Context**: [Describe your current project structure and requirements]

**Goal**: [What you want to implement]

**Reference Example**: [Link to example repository]

**Instructions for AI Agent**:
1. Read and analyze the example repository
2. Identify the core components of the example
3. Adapt the example to my project's:
   - Framework/language
   - Existing architecture
   - Coding conventions
4. Implement the solution while preserving its key characteristics
5. Document any deviations from the reference example and why
```

### Iterative Refinement

Examples evolve as you discover edge cases and improvements:

1. **Initial implementation**: Create basic working example
2. **Documentation**: Document what works and why
3. **Refinement**: Add edge cases and variations discovered in practice
4. **Generalization**: Abstract project-specific details
5. **Versioning**: Track major changes

## The Future: Example Libraries as Model Training Data

Currently, AI agents benefit from example repositories through direct reference during development. Looking forward, these repositories serve another purpose: training data for future models.

### Today: Explicit Reference

AI agents read example repositories in real-time to understand working implementations. This is necessary because current models may not have seen these specific integration patterns during training.

### Tomorrow: Implicit Knowledge

As models are trained on more example repositories and their associated documentation, they'll develop innate understanding of these patterns. The models will "know" that certain configurations work together without needing explicit examples.

### Building for Both

Example repositories should be designed for both uses:

- **Structured format**: Consistent organization makes patterns easier to parse
- **Clear explanations**: Documentation helps both humans and AI understand the "why"
- **Working code**: Executable examples verify the pattern actually works
- **Metadata**: Tags, categories, and dependencies help discovery and training

## Example Categories

Different types of examples serve different purposes:

### Integration Patterns
- Connecting multiple tools/services
- Authentication and authorization flows
- API client implementations
- Third-party service configurations

### Architecture Patterns
- Component organization
- State management approaches
- Module boundaries
- Testing strategies

### Configuration Patterns
- Build system setups
- Development environment configurations
- Deployment pipelines
- Monitoring and observability

### Workflow Patterns
- Development processes
- Code review procedures
- Release management
- Team collaboration

## Best Practices

### Keep Examples Focused

Each repository should demonstrate one primary concept. If you find yourself documenting multiple unrelated patterns, split them into separate example repositories.

### Maintain Working State

Example repositories lose value if they become outdated. Regularly:
- Test that examples still run
- Update dependencies
- Document version compatibility
- Archive examples that are no longer relevant

### Provide Context

Explain not just how to implement the solution, but:
- When to use this approach vs. alternatives
- What problems it solves
- What tradeoffs are involved
- What prerequisites are required

### Show Variations

Include examples of common variations:
- Different frameworks
- Alternative configurations
- Scaled-up versions
- Simplified versions for specific use cases

## Community Example Libraries

Beyond individual repositories, consider contributing to or creating shared example libraries:

### Benefits of Shared Libraries
- Faster discovery of solutions
- Crowdsourced improvements
- Standardized approaches
- Reduced duplication of effort

### Contributing Examples
When sharing examples publicly:
- Use permissive licenses (MIT, Apache 2.0)
- Include comprehensive documentation
- Provide working examples
- Accept community contributions
- Maintain backward compatibility when possible

## Implementing Example-Driven Development in Your Workflow

### Step 1: Start Small
Begin with one complex integration you recently completed. Extract it to a minimal working example and document the key decisions.

### Step 2: Build an Example Library
Create a dedicated space (GitHub organization, monorepo, or documentation site) for collecting examples.

### Step 3: Establish Standards
Define what makes a good example in your context:
- Minimum documentation requirements
- Code quality standards
- Testing expectations
- Update frequency

### Step 4: Train Your Team
Help team members understand:
- When to create new examples
- How to reference existing examples
- How to work with AI agents using examples
- When to update vs. create new examples

### Step 5: Integrate with AI Workflow
Make example repositories a standard part of your AI-assisted development:
- Reference examples in project documentation
- Include example links in CLAUDE.md or similar agent instructions
- Create shortcuts or templates for common example references

## Example: Adding Example-Driven Instructions to CLAUDE.md

```markdown
## Example Repositories

When implementing the following features, reference these example repositories:

### Remote Debugging
- Example: [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug)
- Use for: Setting up debugging in Kubernetes environments
- Adapt for: Our specific container images and namespace structure

### UI Prototyping
- Example: [rapid-ui-prototype-example](https://github.com/nathanfox/rapid-ui-prototype-example)
- Use for: Creating prototype components with mock data
- Adapt for: Our React/TypeScript stack instead of Vue.js

### Example Usage Instructions
1. Read the entire example repository including documentation
2. Identify core components that must be preserved
3. Adapt framework-specific code to our stack
4. Maintain the architectural principles of the example
5. Document any deviations and reasoning
```

## Conclusion

Example-driven development bridges the gap between AI capabilities and proven solutions. By capturing working implementations in well-documented repositories, we create a knowledge base that both humans and AI agents can leverage.

This approach provides immediate value by reducing implementation time and errors. It also contributes to the longer-term goal of training AI models on proven patterns, eventually making explicit reference unnecessary as models internalize these best practices.

The key is recognizing that once you've solved a complex problem, that solution has value beyond your current project. By investing time in extracting, documenting, and sharing examples, you create reusable assets that compound in value with each use.

Start building your example library today. Your future self—and your AI agents—will thank you.

## Next Steps

1. **Identify one complex implementation** from a recent project
2. **Extract it to a minimal repository** with working code
3. **Document the example** with problem, solution, and usage instructions
4. **Test with an AI agent** by referencing it in a new implementation
5. **Iterate and improve** based on actual usage
6. **Share publicly** if appropriate to benefit the broader community

Example-driven development isn't just about documenting code—it's about creating a systematic approach to capturing and reusing knowledge in the age of AI-assisted development.
