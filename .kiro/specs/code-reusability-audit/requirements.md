# Requirements Document: Code Reusability & DRY Principles

## Introduction

This document establishes requirements for maintaining DRY (Don't Repeat Yourself) coding principles across the Fortune Leo application. The system currently has multiple quiz-like flows (distress quiz, investor quiz, partner intake) that share common patterns but have duplicated implementations.

## Glossary

- **DRY (Don't Repeat Yourself)**: A software development principle stating that every piece of knowledge must have a single, unambiguous, authoritative representation within a system
- **Quiz Flow**: Any multi-step form or questionnaire in the application (distress quiz, investor quiz, partner intake)
- **Shared Component**: A reusable UI component that can be used across multiple features
- **Base Logic**: Core functionality that can be extended or configured for specific use cases
- **System**: The Fortune Leo web application

## Requirements

### Requirement 1: Component Reusability

**User Story:** As a developer, I want to identify and use shared components across all quiz-like flows, so that I maintain consistency and reduce code duplication.

#### Acceptance Criteria

1. WHEN implementing a new UI component, THE System SHALL check for existing similar components before creating new ones
2. WHEN a component serves a generic purpose (progress bars, form inputs, navigation), THE System SHALL use or create a shared component in a common directory
3. WHEN multiple flows use similar UI patterns, THE System SHALL extract common components into a shared location
4. WHERE a component is flow-specific, THE System SHALL document why it cannot be generalized
5. WHEN updating a shared component, THE System SHALL verify all usages remain functional

### Requirement 2: Logic Abstraction

**User Story:** As a developer, I want to abstract common business logic into reusable utilities, so that I can maintain consistent behavior across features.

#### Acceptance Criteria

1. WHEN implementing validation logic, THE System SHALL use shared validation utilities for common patterns
2. WHEN handling form state management, THE System SHALL use consistent patterns across all flows
3. WHEN processing user input, THE System SHALL use shared transformation utilities
4. WHERE business logic differs between flows, THE System SHALL use configuration or strategy patterns rather than duplication
5. WHEN conditional logic is needed, THE System SHALL use shared conditional logic utilities

### Requirement 3: Type Safety and Consistency

**User Story:** As a developer, I want consistent type definitions across similar features, so that I can ensure type safety and reduce errors.

#### Acceptance Criteria

1. WHEN defining data structures for quiz-like flows, THE System SHALL use shared base types
2. WHEN extending base types for specific flows, THE System SHALL use TypeScript extension patterns
3. WHEN multiple flows share data fields, THE System SHALL define those fields in a common type
4. WHERE flow-specific types are needed, THE System SHALL extend from base types rather than duplicate
5. WHEN updating shared types, THE System SHALL verify all dependent code compiles successfully

### Requirement 4: Development Workflow Integration

**User Story:** As a developer, I want automated checks and guidelines that remind me to follow DRY principles, so that I maintain code quality consistently.

#### Acceptance Criteria

1. WHEN starting a new feature, THE System SHALL provide steering rules that prompt checking for existing implementations
2. WHEN implementing UI components, THE System SHALL suggest reviewing existing component library
3. WHEN writing business logic, THE System SHALL prompt checking for existing utilities
4. WHERE duplication is necessary, THE System SHALL require documentation of the reasoning
5. WHEN code review occurs, THE System SHALL highlight potential duplication opportunities

### Requirement 5: Documentation and Discovery

**User Story:** As a developer, I want clear documentation of shared components and utilities, so that I can easily discover and use existing code.

#### Acceptance Criteria

1. WHEN shared components exist, THE System SHALL maintain an up-to-date component inventory
2. WHEN shared utilities exist, THE System SHALL document their purpose and usage
3. WHEN patterns are established, THE System SHALL document them for future reference
4. WHERE multiple approaches exist, THE System SHALL document when to use each
5. WHEN new shared code is created, THE System SHALL update the inventory immediately

### Requirement 6: Refactoring and Consolidation

**User Story:** As a developer, I want to identify and consolidate duplicate code, so that I can improve maintainability over time.

#### Acceptance Criteria

1. WHEN duplicate code is identified, THE System SHALL create tasks to consolidate it
2. WHEN consolidating code, THE System SHALL ensure all existing functionality is preserved
3. WHEN refactoring shared code, THE System SHALL run all affected tests
4. WHERE consolidation risks breaking changes, THE System SHALL use feature flags or gradual migration
5. WHEN consolidation is complete, THE System SHALL remove the duplicate implementations
