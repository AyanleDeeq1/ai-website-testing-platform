
# Domain Model

This document describes the domain model of the
AI Website Testing Platform.

## UML Class Diagram

![Domain Model](images/domain-diagram.png)

## Overview

The system contains 10 core entities and 6 enums.

ProjectMember manages project-based user roles.

TestCase supports dependencies between tests.

TestRun, TestResult, and TestArtifact represent
test execution and its results.

## Business Rules

- Each project must have at least one OWNER.
- Users can have different roles in different projects.
- Test cases cannot have circular dependencies.
- Test steps execute in sequence.
