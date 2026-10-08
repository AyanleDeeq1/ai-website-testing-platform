
# Database Design

This document describes the MySQL database design
of the AI Website Testing Platform.

## Database Diagram

![Database Design](images/database-diagram.png)

## Overview

MySQL stores users, projects, memberships,
test cases, test runs, and test results.

MinIO stores uploaded documents and test artifacts.

Redis Streams handles background test jobs.

## Database Constraints

- User email must be unique.
- Each user can have only one membership per project.
- Test steps must have unique sequence numbers
  within a test case.
- Test case dependencies are optional.
- Enum values are stored as VARCHAR strings
  with validation constraints.
