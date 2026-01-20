# Project CRUD With Role-Based Access Control (RBAC)

## Overview

This project implements a CRUD API for Projects with 3 user roles and strict access control rules.

The goal is to understand:
- Role-based authorization
- Ownership-based access
- Clean API architecture
- Real-world CRUD patterns

---

## User Roles

| Role | Description |
|-----|------------|
| admin | Full access to all resources |
| manager | Can manage all projects but cannot delete |
| user | Can manage only own projects |

---

## Database Models

### User Table

| Field | Type | Notes |
|-----|------|------|
| id | UUID | Primary key |
| email | String | Unique |
| password_hash | String | Hashed password |
| role | Enum | admin, manager, user |
| is_active | Boolean | Default true |
| created_at | Timestamp | Auto |
| updated_at | Timestamp | Auto |

---

### Project Table

| Field | Type | Notes |
|-----|------|------|
| id | UUID | Primary key |
| title | String | Required |
| description | Text | Optional |
| status | Enum | draft, active, archived |
| owner_id | UUID | FK → users.id |
| is_deleted | Boolean | Soft delete |
| created_at | Timestamp | Auto |
| updated_at | Timestamp | Auto |

---

## API Base Path

/api/v1

---

## Authentication

All endpoints require authentication via JWT access token.

---

## Endpoints

### Create Project

POST /api/v1/projects

Request Body:
{
  "title": "Project Alpha",
  "description": "Internal research project"
}

Rules:
- owner_id is always the authenticated user
- status defaults to draft

---

### List Projects

GET /api/v1/projects

Query Parameters:
- limit (default 10)
- offset (default 0)
- status (optional)

Behavior:
- admin, manager: all projects
- user: only own projects

---

### Get Project By ID

GET /api/v1/projects/{project_id}

Rules:
- 404 if deleted
- 403 if not owner (user role)

---

### Update Project

PUT /api/v1/projects/{project_id}

Request Body:
{
  "title": "Updated title",
  "description": "Updated description",
  "status": "active"
}

Allowed transitions:
- draft → active
- active → archived

---

### Delete Project (Soft Delete)

DELETE /api/v1/projects/{project_id}

Allowed Roles:
- admin only

---

## Authorization Summary

| Action | Admin | Manager | User |
|------|-------|---------|------|
| Create | Yes | Yes | Yes |
| List All | Yes | Yes | No |
| Read Any | Yes | Yes | No |
| Update Any | Yes | Yes | No |
| Delete | Yes | No | No |

---

## Error Handling

- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 422 Validation Error
- 400 Invalid State Transition

---

## Project Structure

app/
 ├── api/v1/routes/projects.py
 ├── models/
 ├── schemas/
 ├── services/
 └── core/permissions.py

Rules:
- No business logic in routes
- Authorization handled via dependencies and services
