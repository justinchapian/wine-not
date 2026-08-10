---
title: LearnUpon API Documentation
intent: external-api-reference
area: docs
---
# LearnUpon API Documentation

> Source: [https://docs.learnupon.com/api/](https://docs.learnupon.com/api/)

## Table of Contents

- [Overview](#overview)
- [Getting Started](#getting-started)
- [API Access and Authentication](#api-access-and-authentication)
- [Data Pagination](#data-pagination)
- [API Throttling and Rate Limits](#api-throttling-and-rate-limits)
- [Usage Patterns](#usage-patterns)
- [Sample API Calls](#sample-api-calls)
- [Users](#users)
- [Portal Invites](#portal-invites)
- [Enrollments](#enrollments)
- [Exams and Surveys](#exams-and-surveys)
- [MarkCompletes](#markcompletes)
- [Assignments and Assignment Answers](#assignments-and-assignment-answers)
- [Courses](#courses)
- [Modules](#modules)
- [Resources](#resources)
- [Licenses for Courses](#licenses-for-courses)
- [Live Learning](#live-learning)
- [Groups](#groups)
- [Portals](#portals)
- [Group Memberships](#group-memberships)
- [Group Invites](#group-invites)
- [Group Invite Course Enrollment](#group-invite-course-enrollment)
- [Group Courses](#group-courses)
- [Group Managers](#group-managers)
- [Course Instructors](#course-instructors)
- [Learning Paths](#learning-paths)
- [Learning Path Enrollments](#learning-path-enrollments)
- [Gamification](#gamification)
- [Audit Trails](#audit-trails)
- [Certifications](#certifications)
- [Batch User Upload via SFTP](#batch-user-upload-via-sftp)
- [Bulk Operations](#bulk-operations)
- [eCommerce](#ecommerce)
- [Appendix: Language Codes](#appendix-language-codes)
- [Appendix: Timezone Names](#appendix-timezone-names)

---

## Overview

The LearnUpon API (Application Programming Interface) connects to LearnUpon's system to exchange data and support business workflows. As a learning management system (LMS), LearnUpon integrates with existing software to train employees, partners, and customers.

The LearnUpon API can speed up processes by automating time-consuming manual steps. It lets you create and add users to groups, enroll them in courses, and send data back to your 3rd party system.

The LearnUpon API is **RESTful**. Customers access it via Basic HTTP authentication over HTTPS/TLS.

---

## Getting Started

To get the most from this guide, you need to know about:

- JSON API development
- Using a framework/SDK for your preferred language
- Encoding and decoding JSON data
- General HTTP request types
- HTTP response codes

---

## API Access and Authentication

The full URL for accessing the API is:

```
https://yourdomain.learnupon.com/api/v1/resource
```

- `yourdomain` represents your own unique subdomain/portal in LearnUpon
- `resource` represents the path to the resource you are accessing

To access the API, companies must authorize themselves via **Basic HTTP Authentication** over HTTPS/SSL.

### Get Your API Keys

1. Log in to your LearnUpon portal as an admin
2. Select **Settings > Integrations > API Keys**
3. Select **Generate New Keys**

> If you do not see the API Keys option under Integrations, contact the LearnUpon Support team.

**Example API keys** (decommissioned):
- Username: `988d4f1313f881e5ac6bfdfc7f54244aab`
- Password: `905a12r3a0c`

---

## Data Pagination

All API endpoints which list data (such as `/enrollments` or `/users`) use data pagination. The returned data lists **500 records per page**.

To specify a page of data to return, use the `page` attribute:

```
/users?page=3
```

### Pagination Response Headers

| Header Name | Type | Description |
|---|---|---|
| `LU-Current-Page` | integer | The current page of data returned |
| `LU-Records-Per-Page` | integer | Default: 500. Number of records per page |
| `LU-Has-Next-Page` | boolean | Indicates if more pages exist after current page |

---

## API Throttling and Rate Limits

LearnUpon uses rate limiting to prevent abuse. Rate limits apply **per portal** (not per realm).

Both API and OAuth requests count towards the limit per portal.

### Rate Limit Headers

| Header Name | Type | Description |
|---|---|---|
| `X-LU-Rate-Limit-Remaining-Minute` | integer | Remaining API requests for the current minute |
| `X-LU-Rate-Limit-Remaining-Week` | integer | Remaining API requests for the rest of the week (rolling basis) |

---

## Usage Patterns

### Bursty Integrations

If your integration occasionally creates several hundred users and enrollments, consider prioritizing urgent functions and throttling everything else.

**Batch processing (not urgent):** Introduce a "sleep" between requests. Example: run 5 requests, wait 3 seconds, run the next 5.

**Urgent bulk actions:** Split API calls into categories of urgency:
1. Create the client portal
2. Add the admin user
3. Clone courses as next priority
4. Add all other users over time

---

## Sample API Calls

- The API assumes all data sent is in **JSON format** (except GET requests which use Query String parameters)
- All responses are delivered in **JSON format**
- All requests assume content type of `application/json`
- All date/time parameters use UTC format: `YYYY-MM-DDTHH:MI:SSZ`
- URL encode data for GET commands before reaching the API

### cURL on Windows Note

Some versions of cURL on Windows have quoting issues. Swap single/double quotes:

```bash
# Standard
curl .. -d '{"key":"value"}'

# Windows workaround
curl .. -d "{'key':'value'}"
```

### JSON and UTF-8 Syntax

For special characters in user data (apostrophes, fadas), ensure proper commenting:

```bash
# This generates a 400 error
"James O'Connor"

# This works in cURL
"James O'\''Connor"
```

---

## Users

The users resource lets you create, search for (retrieve, find, or list), update and delete users.

### Unique Identifiers: Email or Username

LearnUpon uses a user's `email` attribute as a default unique identifier. You can set up `username` as an alternate unique identifier.

Username requirements:
- 6-30 characters by default (can request 3-character minimum)
- Any Unicode alphanumeric character
- Can use `+ - . , $ # @ _`

### User Timezones

The portal's default timezone applies to user profiles. Users can change their timezone manually. You can also set `timezone_id` when creating or updating a user via API.

### Methods: Users

| Method | Description |
|---|---|
| `GET /users` | Lists all users on the portal |
| `GET /users?version_id=1.1` | Lists all users plus `updated_by` date |
| `GET /users?version_id=1.2` | Lists all users plus `timezone_id` |
| `GET /users/{id}` | Searches by id (includes custom user data) |
| `GET /users/search?email=` | Searches by email (includes custom user data) |
| `GET /users/search?username=` | Searches by username (includes custom user data) |
| `GET /users/customuserdata` | Lists all custom user data fields |
| `POST /users` | Creates a user (minimum: email or username) |
| `PUT /users/` | Updates a user (requires id or email) |
| `DELETE /users/{id}` | Deletes a user account |

### Search for Users

```bash
# Search for all users
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/users

# Search by user_id
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/users/123123

# Search by email
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/users/search?email=learnuponapi@samplelearningco.com

# Search by username
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/users/search?username=learnupon
```

**Sample response (by email):**
```json
{"user":[{"id":662,"first_name":"Learn","last_name":"Upon","email":"learnuponapi@samplelearningco.com",
"created_at":"2012-12-18T15:30:09Z","locale":"de"}]}
```

### Mandatory Parameters for Search

| Parameter | Type | Description |
|---|---|---|
| `email` | string | Email address of the user (default unique identifier) |
| `username` | string | Username of the user (alternate unique identifier) |

### Optional Parameters for Search

| Parameter | Type | Description |
|---|---|---|
| `version_id=1.1` | string | Includes `updated_at` in user object attributes |
| `version_id=1.2` | string | Includes `timezone_id` in user object attributes |
| `login_enabled` | boolean | `true` = enabled users only, `false` = disabled users only |

### User Object Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Unique numeric identifier for the user |
| `last_name` | string | Last name of the user |
| `first_name` | string | First name of the user |
| `email` | string | Email address of the user |
| `username` | string | Only if username is set as unique identifier |
| `locale` | string | ISO language code for the user |
| `created_at` | date/time | Date the user was created (UTC) |
| `updated_at` | date/time | Date of recent change (UTC) |
| `last_sign_in_at` | date/time | Date the user last signed in (UTC) |
| `account_expires` | date/time | Account expiration date (format: `yyyy-mm-dd`) |
| `sign_in_count` | integer | Number of times user logged in |
| `enabled` | boolean | Whether the account is enabled (default: true) |
| `timezone_id` | string | User's timezone for courses/sessions |
| `user_type` | string | One of: `learner`, `instructor`, `manager`, `admin` |
| `can_enroll` | boolean | Manager only: can enroll learners (default: true) |
| `can_unenroll_users` | boolean | Manager only: can unenroll learners (default: false) |
| `can_mark_complete` | boolean | Manager only: can mark complete (default: false) |
| `can_move_groups` | boolean | Manager only: can move users between groups (default: false) |
| `can_act_as_instructor` | boolean | Manager only: has instructor permissions (default: false) |
| `can_manage_trainings` | boolean | Instructor only: can create/edit live learning events (default: false) |
| `can_manage_sessions` | boolean | Instructor only: can create/edit sessions (default: false) |
| `tutor_can_edit_their_courses` | boolean | Instructor only: can edit courses they instruct (default: true) |
| `tutor_can_create_courses` | boolean | Instructor only: can create own courses (default: false) |
| `number_of_enrollments` | integer | Number of enrollments for the user |
| `number_of_enrollments_accessed` | integer | Enrollments in progress/completed/passed |
| `CustomData` | object | Custom user data fields |
| `membership_type` | string | Associations feature: membership type |
| `number_of_points` | integer | Gamification: total points |
| `number_of_badges` | integer | Gamification: total badges |
| `sf_contact_id` | string | Salesforce Contact ID |
| `sf_user_id` | string | Salesforce User ID |
| `is_salesforce_contact` | boolean | Whether user is a Salesforce Contact |
| `customDataFieldValues` | object | Custom user data details |

### customDataFieldValues Object Attributes

| Attribute | Type | Description |
|---|---|---|
| `value` | string/decimal/integer/string_choice/decimal_choice/integer_choice/date | Custom user data value |
| `definition_id` | integer | Unique identifier for the custom user data field |

### CustomDataFieldDefinitions Object

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Unique identifier for the custom field |
| `type_id` | integer | Numeric value identifying the field type |
| `label` | string | Name of the custom field |
| `predefined_values` | string | Optional dropdown values |

### type_id Attribute Definitions

| type_id | Data Type | Example |
|---|---|---|
| 1 | string | `"my_string"` |
| 2 | decimal | `1.00` |
| 3 | integer | `1` |
| 4 | string_choice | `"Monday","Tuesday","Wednesday"` |
| 5 | decimal_choice | `1.00, 2.00, 3.00` |
| 6 | integer_choice | `1,2,3,4,5` |
| 7 | date | `"2020-05-07"` |

### Awards Object (returned from user_id search)

| Attribute | Type | Description |
|---|---|---|
| `number_of_points` | integer | Total points earned |
| `number_of_badges` | integer | Total badges earned |
| `awards` | array | List of badges awarded |

### Awards Data Attributes

| Attribute | Type | Description |
|---|---|---|
| `badge_id` | integer | Unique identifier of the badge |
| `badge_name` | string | Name of the badge |
| `badge_count` | integer | Times user received the badge |
| `total_points` | integer | Total points received with badge |

### Create a User

```bash
# Create user with basic info
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"User": {"last_name":"Upon","first_name":"Learn","email":"learnuponapi@samplelearningco.com","password":"password1","language":"en"}}' \
  https://yourdomain.learnupon.com/api/v1/users

# Create user with manager+instructor permissions
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"User": {"last_name":"Upon","first_name":"Learn","email":"learnuponapi@luptest.com","password":"password1","language":"en","user_type":"manager","can_act_as_instructor":"true"}}' \
  https://yourdomain.learnupon.com/api/v1/users
```

#### Mandatory Parameters for Creating a User

| Parameter | Type | Description |
|---|---|---|
| `email` | string | Email address (default unique identifier) |
| `username` | string | Username (alternative to email) |
| `password` | string | Password, 6+ characters |

#### Optional Parameters for Creating a User

| Parameter | Type | Description |
|---|---|---|
| `last_name` | string | Last name |
| `first_name` | string | First name |
| `language` | string | ISO language code |
| `CustomData` | object | Custom user data fields |
| `account_expires` | date/string | Account expiration (format: `yyyy-mm-dd`) |
| `enabled` | boolean | Account enabled (default: true) |
| `user_type` | string | One of: `learner`, `instructor`, `manager`, `admin` |
| `timezone_id` | string | User's timezone |
| `can_enroll` | boolean | Manager: can enroll learners (default: true) |
| `can_unenroll_users` | boolean | Manager: can unenroll (default: false) |
| `can_mark_complete` | boolean | Manager: can mark complete (default: false) |
| `can_move_groups` | boolean | Manager: can move users (default: false) |
| `can_act_as_instructor` | boolean | Manager: instructor permissions (default: false) |
| `can_manage_trainings` | boolean | Instructor: create/edit events (default: false) |
| `can_manage_sessions` | boolean | Instructor: create/edit sessions (default: false) |
| `tutor_can_edit_their_courses` | boolean | Instructor: edit courses (default: true) |
| `tutor_can_create_courses` | boolean | Instructor: create courses (default: false) |
| `change_password_on_first_login` | boolean | Force password change (default: false) |
| `membership_type` | string | Associations: `Member` or `Non-member` |
| `sf_contact_id` | string | Salesforce Contact ID |
| `sf_user_id` | string | Salesforce User ID |

#### Data Returned After Creating a User

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Unique identifier for the created user |

### Update a User

```bash
# Update user by user_id
curl -X PUT -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"User": {"last_name":"Upon","first_name":"Learn","email":"learnuponapi@samplelearningco.com","password":"password1","language":"en"}}' \
  https://yourdomain.learnupon.com/api/v1/users/12345
```

**Update rules:**
- Find user using `user_id` (returned when creating a user), OR
- Enter `0` as user_id but include `email` (or `username`) in the JSON payload
- To update email/username without numeric id: use existing email/username with id of 0, and specify `new_email` or `new_username`

#### Optional Parameters for Updating a User

Same as creating a user, plus:

| Parameter | Type | Description |
|---|---|---|
| `new_email` | string | New email address |
| `new_username` | string | New username |

### Delete a User

```bash
curl -X DELETE -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/users/12345
```

| Parameter | Type | Description |
|---|---|---|
| `id` | integer | Unique identifier of the user to delete |

### Update Custom User Data

```bash
curl -X PUT -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"User": {"CustomData":{"Department":"Sales"},"user_type":"learner","last_name":"TestUserLastName","first_name":"TestUserFirstName","email":"testuser@samplelearningco.com","password":"123123"}}' \
  https://yourdomain.learnupon.com/api/v1/users/12345
```

Two ways to send custom user data:

1. **Use definition label as key:**
```json
"CustomData": {"department":"Support","position":"Account Manager"}
```

2. **Use definition type_id as key (with `use_definition_ids` flag):**
```json
"CustomData": {"use_definition_ids":1,"231":"Support","640":"Account Manager"}
```

---

## Portal Invites

The `portal_invite` resource lets you invite a user into a portal or group, including user attributes such as first and last name.

### Methods: Portal Invites

| Method | Description |
|---|---|
| `POST /portal_invite` | Invite individual users to join a portal or group |

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"Invite": {"email":"someone@samplelearningco.com","first_name":"Learn","last_name":"Upon"}}' \
  https://yourdomain.learnupon.com/api/v1/portal_invite
```

### Mandatory Parameters

| Parameter | Type | Description |
|---|---|---|
| `email` | string | Email of the user |
| `username` | string | Username (if enabled as unique identifier) |

### Optional Attributes

| Attribute | Type | Description |
|---|---|---|
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `expires_at` | date/string | Account expiration date |
| `group_id` | integer | Group to add user to on acceptance |
| `CustomData` | object | Custom user data |
| `sf_contact_id` | string | Salesforce Contact ID |
| `sf_user_id` | string | Salesforce User ID |
| `portal_membership_type_id` | integer | User permissions: 1=learner, 2=admin, 3=instructor, 4=manager |

### Attributes Returned

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Unique identifier for the portal invite |
| `email` | string | Email where invite was sent |

---

## Enrollments

The enrollments resource lets you create new enrollments, search for existing enrollments, and delete (unenroll) learners from courses.

### Methods: Enrollments

| Method | Description |
|---|---|
| `POST /enrollments` | Creates an enrollment onto a course |
| `PATCH /enrollments/{id}` | Update enrollment details (requires enrollment id) |
| `GET /enrollments/{id}` | Search by enrollment id |
| `GET /enrollments/{id}/modules` | Search for modules by enrollment id |
| `GET /enrollments/search?user_id=` | Search by user_id |
| `GET /enrollments/search?email=` | Search by email |
| `GET /enrollments/search?username=` | Search by username |
| `GET /enrollments/search?course_name=` | Search by course_name |
| `GET /enrollments/search?course_id=` | Search by course_id |
| `DELETE /enrollments/{enrollment_id}` | Delete enrollment |

### Create an Enrollment

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"Enrollment": {"email":"learnuponapi@samplelearningco.com","course_name":"Hello API"}}' \
  https://yourdomain.learnupon.com/api/v1/enrollments
```

#### Mandatory Parameters

| Parameter | Type | Description |
|---|---|---|
| `email` | string | Email of the user |
| `username` | string | Username of the user |
| `course_name` | string | Exact title of the course |
| `course_id` | integer | Unique identifier for the course |

#### Optional Parameters

| Parameter | Type | Description |
|---|---|---|
| `re_enroll_if_completed` | boolean | Re-enroll if previously completed |
| `due_date` | date/time | Due date (format: `YYYY-MM-DD`) |
| `expires_at` | date/time | Expiry date (format: `YYYY-MM-DD`) |

#### Data Returned

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Unique identifier for the enrollment |
| `created_at` | date/time | Date created (UTC) |
| `user_id` | integer | User identifier |

### Update an Enrollment

```bash
curl -X PATCH -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"due_date":"2025-12-01","expiry_date":"2026-04-11"}' \
  https://yourdomain.learnupon.com/api/v1/enrollments/345678
```

| Parameter | Type | Description |
|---|---|---|
| `due_date` | date/time | Due date (format: `YYYY-MM-DD`) |
| `expires_at` | date/time | Expiry date (format: `YYYY-MM-DD`) |

### Search for Enrollments

```bash
# By enrollment_id
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/enrollments/2757

# By user_id with date filter
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/enrollments/search?user_id=1234&date_from=2017-11-22&date_to=2017-11-22

# By course_name with pagination
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/enrollments/search?course_name=Hello%20API&page=3
```

#### Optional Search Parameters

| Parameter | Type | Description |
|---|---|---|
| `date_from` | date/time | Start date filter (format: `YYYY-MM-DD`) |
| `date_to` | date/time | End date filter (format: `YYYY-MM-DD`) |
| `modules` | string | Returns module list (requires enrollment_id) |

### Enrollment Data Attributes Returned

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Unique enrollment identifier |
| `percentage` | integer | Score achieved |
| `date_started` | date/time | Date user started (UTC) |
| `date_completed` | date/time | Date user finished (UTC) |
| `date_lastaccessed` | date/time | Date last accessed (UTC) |
| `date_enrolled` | date/time | Date enrolled (UTC) |
| `course_id` | integer | Course identifier |
| `course_name` | string | Course title |
| `course_source_id` | integer | Source course_id for re-versioned courses |
| `user_id` | integer | User identifier |
| `first_name` | string | User first name |
| `last_name` | string | User last name |
| `email` | string | User email |
| `username` | string | User username (if enabled) |
| `due_date` | date/time | Due date (UTC) |
| `calculated_due_date` | date/time | Earliest due date from multiple sources |
| `status` | string | One of: `not_started`, `in_progress`, `completed`, `passed`, `failed`, `pending_review` |
| `course_access_expires_at` | date/time | Course access expiration (UTC) |
| `certificate_name` | string | Certificate name |
| `cert_expires_at` | date/time | Certificate expiration |
| `version` | integer | Course version |
| `was_recertified` | boolean | Whether recertification applied |
| `percentage_complete` | integer | Modules completed as percentage |
| `unenrolled` | boolean | Whether enrollment was unenrolled |
| `from_store` | boolean | Enrollment via eCommerce (default: false) |
| `from_catalog` | boolean | Enrollment via catalog (default: false) |
| `updated_at` | date/time | Last updated (UTC) |
| `group_id` | integer | Learner's group_id |
| `is_overdue` | boolean | Whether course is overdue |
| `modules` | array | Module details for enrollment |

### Module Data Attributes (by enrollment_id)

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Module identifier |
| `title` | string | Module name |
| `sequence` | integer | Module order |
| `date_started` | date/time | Date started (UTC) |
| `date_completed` | date/time | Date completed (UTC) |
| `date_lastaccessed` | date/time | Date last accessed (UTC) |
| `status` | string | One of: `not_started`, `in_progress`, `completed`, `passed`, `failed`, `pending_review` |

### Delete Enrollments

Deleting an enrollment removes the connection between user and course â€” it does not delete the user or course.

```bash
curl -X DELETE -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"remove_from_history":"true"}' \
  https://yourdomain.learnupon.com/api/v1/enrollments/12345
```

| Parameter | Type | Description |
|---|---|---|
| `id` | integer | Enrollment identifier (mandatory) |
| `remove_from_history` | boolean | Force-delete including completed enrollments (optional) |

---

## Exams and Surveys

The exams resource lets you search for exam/survey settings, questions, answers, and enrollment data. Surveys use the same resource as exams with a unique `exam_id`.

### Methods: Exams and Surveys

| Method | Description |
|---|---|
| `GET /exams/{exam_id}` | Exam/survey settings |
| `GET /exams/{exam_id}/questions` | Questions and answer options |
| `GET /exams/{exam_id}/enrollments` | Enrollment data for all users |
| `GET /exams/{exam_id}/answers/{exam_enrollment_id}` | Specific user's answers |

### Exam Settings

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/exams/{exam_id}
```

**Sample response (exam):**
```json
{
    "exam": {
        "name": "Mars exam questions",
        "is_survey": false,
        "exam_id": 110117,
        "is_timed_exam": true,
        "pass_percentage": 50,
        "pass_mark": null,
        "is_knowledge_check": false,
        "attempts": "unlimited",
        "time_for_exam": 2
    }
}
```

#### Exam Attributes

| Attribute | Type | Description |
|---|---|---|
| `exam_id` | integer | Unique identifier for the exam module |
| `name` | string | Exam name |
| `is_survey` | boolean | Whether it's a survey (default: false) |
| `is_knowledge_check` | boolean | Whether it's a knowledge check (default: false) |
| `attempts` | integer | Number of attempts allowed |
| `is_timed_exam` | boolean | Whether exam is timed (default: false) |
| `time_for_exam` | integer | Time allowed (timed exams only) |
| `pass_mark` | integer | Passing mark |
| `pass_percentage` | integer | Passing percentage |

### Exam Questions

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/exams/{exam_id}/questions
```

#### Question Object Attributes

| Attribute | Type | Description |
|---|---|---|
| `question_id` | integer | Unique identifier for the question |
| `question_text` | text | Question wording |
| `question_type` | string | One of: `choice`, `feedback`, `rating`, `order list`, `match list`, `fill in the blanks` |
| `question_choice_type` | string | `single` or `multiple` choice |
| `points` | integer | Points for question |
| `answers` | object | Available answers |

#### Answers Object Attributes

| Attribute | Type | Description |
|---|---|---|
| `answer_id` | integer | Unique answer identifier |
| `answer_text` | string | Answer text |
| `is_correct` | string | For choice questions: `1`=correct, `0`=incorrect |
| `for_matching` | text | For match list: correct answer |
| `sequence` | integer | Answer order |
| `predefined_answers` | text | For fill in blanks: selectable answers |
| `rating_options` | object | For rating questions: rating options |

### Exam Enrollment Data

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/exams/{exam_id}/enrollments
```

**Sample response:**
```json
{
    "exam_enrollments": [
        {
            "exam_id": 110117,
            "exam_enrollment_id": 2590435,
            "status": "Passed",
            "time_taken": 12,
            "attempts_used": 1,
            "score": 100,
            "user_id": 123456,
            "date_completed": "2024-03-22T17:29:13Z"
        }
    ]
}
```

| Attribute | Type | Description |
|---|---|---|
| `exam_id` | integer | Exam module identifier |
| `exam_enrollment_id` | integer | User's enrollment to the exam |
| `user_id` | integer | User identifier (hidden for anonymous surveys) |
| `status` | string | Enrollment status |
| `attempts_used` | integer | Attempts used |
| `time_taken` | float | Time spent (minutes) |
| `score` | integer | User's score (not for surveys) |
| `date_completed` | date/time | Completion date (UTC) |

### Exam Answers by User

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/exams/{exam_id}/answers/{exam_enrollment_id}
```

| Attribute | Type | Description |
|---|---|---|
| `exam_id` | integer | Exam identifier |
| `user` | string | User name (hidden for anonymous surveys) |
| `email` | string | User email (hidden for anonymous surveys) |
| `questions` | object | Questions with user's answers |

#### Questions/Answers Object

| Attribute | Type | Description |
|---|---|---|
| `question_id` | integer | Question identifier |
| `question_text` | text | Question wording |
| `overall_status` | string | `0`=not attempted, `1`=correct, `2`=incorrect |
| `correct_answer` | text | Correct answer (exams only) |
| `given_answers` | text | User's provided answer |

---

## MarkCompletes

The markcompletes resource lets you search for, view and update a learner's enrollment status manually (complete, passed, or failed) without the learner completing the enrollment.

### Methods: MarkCompletes

| Method | Description |
|---|---|
| `GET /markcompletes` | Search for MarkComplete enrollments |
| `POST /markcompletes` | Create MarkComplete enrollments |

### Search for MarkCompletes

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/markcompletes
```

#### Optional Parameters

| Parameter | Type | Description |
|---|---|---|
| `id` | integer | MarkComplete identifier |
| `user_id` | integer | User identifier |
| `enrollment_id` | integer | Enrollment identifier |
| `course_id` | integer | Course identifier |

#### Attributes Returned

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | MarkComplete identifier |
| `created_at` | date/time | Date created (UTC) |
| `date_completed` | date/time | Date enrollment was completed |
| `status` | string | One of: `completed`, `passed`, `failed` |
| `percentage` | integer | Score achieved |
| `user_id` | integer | User who marked complete (NULL if via API) |
| `enrollment_id` | integer | Enrollment identifier |
| `notes` | string | Explanatory notes |
| `action_source_type` | string | One of: `UI`, `API` |

### Create MarkComplete

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"Markcomplete":{"enrollment_id":"12345","date_completed":"2017-07-07T13:00:00Z","status":"passed","percentage":"100"}}' \
  https://yourdomain.learnupon.com/api/v1/markcompletes
```

#### Mandatory Parameters

| Parameter | Type | Description |
|---|---|---|
| `enrollment_id` | integer | Enrollment to mark complete |
| `date_completed` | date/time | Completion date (UTC) |
| `status` | string | One of: `completed`, `passed`, `failed` |
| `percentage` | integer | Score (required for passed/failed) |

#### Optional Parameters

| Parameter | Type | Description |
|---|---|---|
| `notes` | string | 2000 char free text |
| `award_certs` | boolean | Award certificates (default: true) |
| `award_credits` | boolean | Award credits (default: true) |
| `award_badges` | boolean | Award badges (default: true) |

---

## Assignments and Assignment Answers

The assignments resource lets you search for all answers for a specific assignment, or individual answers.

### Methods

| Method | Description |
|---|---|
| `GET /assignments/{assignment_id}/answers` | All answers for a specific assignment |
| `GET /assignments/{assignment_id}/answers/{answer_id}` | Specific answer on an assignment |

> For assignment modules, the `assignment_id` is the same as the `module_id`.

### Find All Answers

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/assignments/{assignment_id}/answers
```

### Assignment Answer Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Unique answer identifier |
| `answer` | string | Text of learner's answer |
| `feedback` | string | Instructor feedback or auto-acknowledgement |
| `score` | integer | Numeric result |
| `status` | string | One of: `not_started`, `in_progress`, `completed`, `passed`, `failed`, `pending_review` |
| `user_id` | integer | Learner identifier |
| `reviewed_at` | date/time | Review date (UTC) |
| `assignment_id` | integer | Assignment identifier (same as module_id) |
| `answer_submitted_at` | date/time | Submission date |
| `was_rejected` | boolean | Whether rejected (default: false) |
| `rejected_at` | date/time | Rejection date (can be null) |
| `was_autocompleted` | boolean | Whether auto-scored (default: false) |
| `reviewed_by_user_id` | integer | Reviewing instructor (can be null) |
| `rejected_by_user_id` | integer | Rejecting instructor (can be null) |

---

## Courses

The courses resource lets you search for, create, update, delete and publish courses. You can also clone courses and add/remove modules.

### Methods: Courses

| Method | Description |
|---|---|
| `GET /courses` | Search for courses |
| `POST /courses` | Create a course |
| `POST /courses/publish` | Publish a course |
| `POST /courses/clone` | Clone a course |
| `POST /courses/add_module` | Add a module to a course |
| `POST /courses/remove_module` | Remove a module from a course |
| `POST /courses/{id}/set_membership_type_prices` | Set prices by membership type |
| `POST /courses/{id}/assign_certificate` | Add certificate to course |
| `PUT /courses/{id}` | Update a course |
| `DELETE /courses/{id}` | Delete a course |

### Search for Courses

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/courses
```

#### Optional Search Parameters

| Parameter | Type | Description |
|---|---|---|
| `name` | string | Course name (pattern matching, case insensitive) |
| `course_id` | integer | Unique course identifier |
| `include_all_versions` | boolean | Include archived versions (default: false) |
| `include_draft` | boolean | Include drafts (default: false) |
| `include_licensed_courses` | boolean | Include licensed courses (default: false) |

### Courses Object Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Unique course identifier |
| `name` | string | Course title |
| `version` | integer | Course version |
| `source_id` | integer | Source course_id (stable across versions) |
| `created_at` | date/time | Creation date (UTC) |
| `sellable` | boolean | Published to store (default: false) |
| `cataloged` | boolean | Published to catalog (default: false) |
| `date_published` | date/time | Publish date (UTC) |
| `keywords` | string | Search keywords |
| `reference_code` | string | Customer reference code |
| `manager_can_enroll` | boolean | Manager can enroll (default: false) |
| `allow_users_rate_course` | boolean | Learners can rate (default: true) |
| `number_of_reviews` | integer | Number of ratings |
| `number_of_stars` | integer | Stars awarded |
| `course_length` | float | Course length |
| `course_length_unit` | string | `hours` or `minutes` |
| `num_enrolled` | integer | Learners enrolled |
| `num_not_started` | integer | Not started count |
| `num_in_progress` | integer | In progress count |
| `num_completed` | integer | Completed count |
| `num_passed` | integer | Passed count |
| `num_failed` | integer | Failed count |
| `num_pending_review` | integer | Pending review count |
| `number_of_modules` | integer | Module count |
| `price` | integer | Price in cents |
| `published_status_id` | string | One of: `published`, `draft`, `archived`, `under_revision` |
| `difficulty_level` | string | One of: `not_applicable`, `basic`, `intermediate`, `advanced` |
| `description_html` | string | Description with HTML |
| `description_text` | string | Description raw text |
| `objectives_html` | string | Objectives with HTML |
| `objectives_text` | string | Objectives raw text |
| `credits_to_be_awarded` | string | Credits list (e.g., `"CEU:2.00,CPE:3.00"`) |
| `due_days_after_enrollment` | integer | Days after enrollment when due |
| `due_date_after_enrollment` | date/time | Fixed due date |
| `thumbnail_image_url` | string | Course thumbnail URL |
| `owner_first_name` | string | Course owner first name |
| `owner_last_name` | string | Course owner last name |
| `owner_email` | string | Course owner email |
| `owner_id` | integer | Course owner identifier |

### Create a Course

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"Course": {"name":"Course name","owner_id":123123}}' \
  https://yourdomain.learnupon.com/api/v1/courses
```

#### Mandatory Parameters

| Parameter | Type | Description |
|---|---|---|
| `name` | string | Course name |
| `owner_id` | integer | Course owner (admin or instructor) |

#### Optional Parameters

| Parameter | Type | Description |
|---|---|---|
| `description` | string | Course description (can include HTML) |
| `objectives` | string | Course objectives (can include HTML) |
| `keywords` | string | Search keywords |
| `reference_code` | string | Customer reference code |
| `allow_users_rate_course` | boolean | Allow ratings (default: true) |
| `review_is_mandatory` | boolean | Mandatory rating (default: false) |
| `difficulty_level` | integer | 0=N/A, 1=Basic, 2=Intermediate, 3=Advanced |
| `course_length` | float | Course length |
| `course_length_unit` | integer | 0=minutes, 1=hours |
| `enable_sequencing` | boolean | Sequential modules (default: false) |
| `manager_can_enroll` | boolean | Manager enrollment (default: false) |
| `allow_relaunch_after_completion` | boolean | Allow relaunch (default: true) |
| `days_after_enrollment` | integer | Access days after enrollment |
| `date_after_enrollment` | date/time | Access until date (format: `mm/dd/yyyy`) |
| `send_completion_email` | integer | 0=Never, 1=On complete/pass, 2=On complete/pass/fail |
| `send_completion_reminders` | boolean | Send reminders (default: false) |
| `completion_reminder_days` | integer | Days for first reminder |
| `sellable` | boolean | Sellable on store (default: false) |
| `price` | float | Price (0.00 for free) |
| `cataloged` | boolean | List in catalog (default: false) |
| `pass_mark` | integer | Pass/fail threshold |
| `due_days_after_enrollment` | integer | Days until due |
| `due_date_after_enrollment` | date/time | Due date (format: `mm/dd/yyyy`) |
| `send_due_date_reminders` | boolean | Due date reminders (default: false) |
| `due_date_reminder_days` | integer | First reminder days before due |
| `due_date_reminder_days_2` | integer | Second reminder days before due |

### Publish a Course

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"course_id":123123}' \
  https://yourdomain.learnupon.com/api/v1/courses/publish
```

#### Optional Publish Parameters (for re-versioning)

| Parameter | Type | Description |
|---|---|---|
| `not_started_logic` | integer | 1=Move to new version, 2=Leave on old (default) |
| `in_progress_logic` | integer | 1=Move to new version, 2=Leave on old (default) |
| `lp_logic` | integer | 1=Replace on learning paths, 2=Leave current (default) |

### Clone a Course

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"clone_to_portal_id":"5127","course_id":"12364","publish_after_clone":"true"}' \
  https://yourdomain.learnupon.com/api/v1/courses/clone
```

#### Clone Parameters

| Parameter | Type | Description |
|---|---|---|
| `course_id` | integer | Course to clone (mandatory) |
| `clone_to_portal_id` | integer | Destination portal (optional) |
| `publish_after_clone` | boolean | Publish after clone (default: false) |
| `owner_id` | integer | New course owner (optional) |

Returns a `guid` for subsequent clones to the same location.

### Add/Remove Module

```bash
# Add module
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"course_id":123123,"module_id":123456}' \
  https://yourdomain.learnupon.com/api/v1/courses/add_module

# Remove module
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"course_id":123123,"module_id":123456}' \
  https://yourdomain.learnupon.com/api/v1/courses/remove_module
```

> Course must be in **draft** state. Re-publish after adding/removing modules.

### Add Certificate to Course

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"certification_guid":"6377075f-4abd-11ef-97d5-06c3fede0100","certification_expires_days":180,"recertify_when_expires":true,"recertify_days_before_expires":30}' \
  https://yourdomain.learnupon.com/api/v1/courses/345678/assign_certificate
```

### Delete a Course

```bash
curl -X DELETE -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/courses/12345
```

> Cannot delete courses with enrolled learners. Unenroll first.

---

## Modules

Modules are the "chunks" of content that make up a course.

### Methods: Modules

| Method | Description |
|---|---|
| `GET /modules` | Search for all modules on portal |
| `GET /modules?course_id=` | Search modules within a specific course |
| `POST /modules` | Publish audio/video modules |
| `POST /modules/clone` | Copy module from one course to another |

### Search for Modules

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/modules?course_id=12531
```

### Module Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Module identifier |
| `title` | string | Module title |
| `tags` | string | Comma-separated tags |
| `created_at` | date/time | Creation date (UTC) |
| `updated_at` | date/time | Updated date (UTC) |
| `exam_id` | integer | Associated exam identifier (can be null) |
| `sequence` | integer | Module order in course |
| `number_of_linked_courses` | integer | Courses containing this module |
| `component_type` | string | One of: `page`, `assignment`, `ilt session`, `exam`, `tincan`, `scorm`, `unknown` |
| `description_html` | text | Description with HTML |
| `description_text` | text | Description plain text |
| `creator_id` | integer | Creator user ID |
| `creator_first_name` | string | Creator first name |
| `creator_last_name` | string | Creator last name |
| `creator_email` | string | Creator email |
| `assignment_passing_percentage` | integer | Assignment pass score |
| `assignment_question_html` | string | Assignment question (HTML) |
| `assignment_question_text` | string | Assignment question (text) |
| `location_id` | integer | ILT location identifier |
| `location` | string | ILT location name |
| `start_at` | date/time | ILT session start (UTC) |
| `end_at` | date/time | ILT session end (UTC) |
| `timezone` | string | ILT location timezone |
| `number_enrolled_on_session` | integer | ILT enrollments |
| `max_capacity` | integer | ILT max capacity |
| `session_tutor_id` | integer | ILT tutor user ID |
| `training_id` | integer | Live learning event identifier |
| `session_id` | integer | Live learning session identifier |

### Create Audio/Video Module

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 93a306ad8fff2fe3dbd1:d670237770e52266234ecd45678bce -d \
  '{"module_title":"Api video module","video_url":"https://url_to_the_video"}' \
  https://yourdomain.learnupon.com/api/v1/modules
```

| Parameter | Type | Description |
|---|---|---|
| `module_title` | string | Title (mandatory) |
| `video_url` | string | URL to video file |
| `audio_url` | string | URL to audio file |
| `tags` | string | Comma-separated tags (optional) |

### Clone a Module

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 93a306ad8fff2fe3dbd1:d670237770e52266234ecd45678bce -d \
  '{"source_course_id":"12345","target_course_id":"67890","module_id":"87654"}' \
  https://yourdomain.learnupon.com/api/v1/modules/clone
```

> Target course must be in draft state. Use `guid` parameter for subsequent clones.

---

## Resources

Resources are files for learners outside of a course, accessible via the LearnUpon library or a direct link.

### Methods: Resources

| Method | Description |
|---|---|
| `GET /resources` | List all resources in portal |
| `POST /resources/{id}/link` | Link resource to sub-portal |
| `DELETE /resources/{id}/link` | Remove linked resource from sub-portal |

### Resource Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Resource identifier |
| `title` | string | Resource name |
| `description` | string | Description with HTML |
| `file_name` | string | Uploaded file name |
| `file_size` | integer | File size (KB) |
| `content_type` | string | Content format |
| `is_visible` | boolean | Available to learners (default: false) |

### Link/Unlink Resources

```bash
# Link to sub-portal
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"link_to_portal_id": 12871}' \
  https://yourdomain.learnupon.com/api/v1/resources/{id}/link

# Remove from sub-portal
curl -X DELETE -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"linked_portal_id": 12871}' \
  https://yourdomain.learnupon.com/api/v1/resources/{id}/link
```

---

## Licenses for Courses

Licenses publish courses from a top-level portal to sub-portals while controlling access.

### Methods: Licenses

| Method | Description |
|---|---|
| `GET /licenses` | Search for course licenses |
| `GET /licenses?course_id={course_id}` | Search licenses for a course |
| `GET /licenses/{license_id}` | Search individual license |
| `POST /licenses` | Create licenses |
| `PUT /licenses/{license_id}` | Update a license |
| `DELETE /licenses/{license_id}` | Delete a license |

### License Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | License identifier (per portal) |
| `course_id` | integer | Course identifier (consistent across portals) |
| `portal_id` | integer | Sub-portal identifier |
| `expiry_date` | date/time | License expiration (can be null) |
| `learner_limit` | integer | Seats allowed (can be null) |
| `created_at` | string | Creation date |
| `updated_at` | string | Last updated date |
| `allow_managers_to_enroll_learners` | boolean | Manager enrollment permission |
| `resale_option` | string | One of: `not_allowed`, `allowed_without_price_change`, `allowed_with_price_change` |

### Create Licenses

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"course_ids":[4754430,4381339],"portal_ids":[16196],"expiry_date":"2027-12-31","learner_limit":50,"allow_managers_to_enroll_learners":true,"resale_option":"allowed_without_price_change"}' \
  https://yourdomain.learnupon.com/api/v1/licenses
```

---

## Live Learning

Live Learning provides containers (events) to hold sessions. Events hold metadata and can have individual sessions or session series.

### Methods: Live Learning Sessions (GET)

| Method | Description |
|---|---|
| `GET /live-learnings/sessions/search?course_id=` | Search sessions by course_id |
| `GET /live-learnings/sessions/search?location_id=` | Search by location_id |
| `GET /live-learnings/sessions/search?instructor_user_id=` | Search by instructor |
| `GET /live-learnings/sessions/search?course_status=` | Search by course_status |
| `GET /live-learnings/sessions/search?start_date=` | Search by start_date |
| `GET /live-learnings/sessions/search?end_date=` | Search by end_date |
| `GET /live-learnings/sessions/search?event_id=` | Search by event_id |
| `GET /live-learnings/sessions/list` | List all sessions |
| `GET /live-learnings/events/{event_id}/sessions/{session_id}` | Session details |
| `GET /live-learnings/sessions/{session_id}/attendance` | Session attendance |
| `GET /live-learnings/custom_session_data_definitions/list` | Custom session data definitions |

### Methods: Live Learning Events (GET)

| Method | Description |
|---|---|
| `GET /live-learnings/events/{event_id}` | Event details with session_series_tags |

### Methods: Session Series (GET)

| Method | Description |
|---|---|
| `GET /live-learnings/session-series/search?course_id=` | Search series by course_id |
| `GET /live-learnings/session-series/{id}` | Search by session_series_id |

### Methods: Learner Sessions (GET)

| Method | Description |
|---|---|
| `GET /live-learnings/learners/{user_id}/sessions` | Learner's scheduled sessions |

### Methods: Events (POST)

| Method | Description |
|---|---|
| `POST /live-learnings/events/` | Create event |
| `POST /live-learnings/events/{event_id}/sessions` | Create sessions within event |
| `POST /live-learnings/sessions/{session_id}/add_to_session` | Add learner to session |
| `POST /live-learnings/events/{event_id}/sessions/{session_id}/mark_attendance` | Mark attendance |
| `POST /live-learnings/events/{event_id}/sessions/{session_id}/cancel_registration` | Cancel registration |
| `POST /live-learnings/events/{event_id}/sessions/{session_id}/cancel` | Cancel session |
| `POST /live-learnings/events/{event_id}/session-series` | Create session series |
| `POST /live-learnings/events/{event_id}/session-series/{id}/part/{session_detail_id}/mark_attendance` | Mark series attendance |
| `POST /live-learnings/events/{event_id}/session-series/{id}/cancel` | Cancel series |

### Methods: Events (PUT)

| Method | Description |
|---|---|
| `PUT /live-learnings/events/{event_id}` | Update event |
| `PUT /live-learnings/events/{event_id}/sessions/{session_id}` | Update session |
| `PUT /live-learnings/events/{event_id}/session-series/{id}` | Update session series |
| `PUT /live-learnings/events/{event_id}/session-series/{id}/part` | Update series part |

### Methods: Metadata

| Method | Description |
|---|---|
| `GET /timezones` | Available timezones |
| `GET /locations` | Available locations |

### Create a Live Learning Event

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"event":{"title":"APIs made easy","owner_user_ids":123456,"waitlist_status":1}}' \
  https://yourdomain.learnupon.com/api/v1/live-learnings/events
```

#### Mandatory Event Parameters

| Parameter | Type | Description |
|---|---|---|
| `title` | string | Event title |
| `owner_user_ids` | array (integer) | Owner user_ids |
| `owner_emails` | array (string) | Owner emails (alternative) |
| `owner_usernames` | array (string) | Owner usernames (alternative) |

#### Optional Event Parameters

| Parameter | Type | Description |
|---|---|---|
| `instructor_user_ids` | array (integer) | Instructor user_ids |
| `waitlist_status` | integer | 0=Disabled (default), 1=Enabled |
| `description` | string | Event description |
| `completion_requirement` | integer | 0=None, 1=Join session (default), 2=Min attendance |
| `minimum_attendance_threshold` | integer | % of session to attend |
| `session_registration_choice` | integer | 0=automatic, 1=manual (default) |
| `learner_can_choose_session` | boolean | Self-enroll (default: false) |
| `learner_can_cancel_registration` | boolean | Cancel allowed (default: false) |
| `session_type` | string | `0`/`individual` or `1`/`series` |
| `visibility` | boolean | 0=all users, 1=owners/instructors only |
| `enabled_notifications` | array (string) | Notification types to enable |
| `session_series_tags` | array | Topics for session series |

### Create Individual Sessions

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"session":{"title":"Advanced APIs, part 1","start_time":"2023-11-04T09:00:00","end_time":"2023-11-05T09:00:00","timezone_id":"Eastern Time (US & Canada)","instructor_user_ids":[111111,222222],"primary_instructor_user_id":222222}}' \
  https://yourdomain.learnupon.com/api/v1/live-learnings/events/361212/sessions
```

#### Mandatory Session Parameters

| Parameter | Type | Description |
|---|---|---|
| `title` | string | Session title |
| `instructor_user_ids` | array (integer) | Instructor user_ids |
| `primary_instructor_user_id` | integer | Primary instructor |
| `start_time` | date/time | Start time (UTC) |
| `end_time` | date/time | End time (UTC) |
| `timezone_id` | string | Session timezone |

#### Optional Session Parameters

| Parameter | Type | Description |
|---|---|---|
| `location_id` | integer | Location for in-person sessions |
| `custom_webinar_url` | string | Webinar URL |
| `minimum_attendance_threshold` | integer | Attendance override |
| `description` | string | Session description |
| `min_capacity` | integer | Minimum capacity |
| `max_capacity` | integer | Maximum capacity |

### Add User to Session

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"course_id":1614047,"user_id":291154}' \
  https://yourdomain.learnupon.com/api/v1/live-learnings/sessions/200039/add_to_session
```

### Mark Attendance

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"attendance":[{"email":"user1@yourcompany.com","completion_status":5},{"email":"user2@yourcompany.com","completion_status":10,"duration":42}]}' \
  https://yourdomain.learnupon.com/api/v1/live-learnings/events/123/sessions/223/mark_attendance
```

- `completion_status`: 5=absent, 10=present
- `duration`: Minutes of attendance (mandatory if min attendance threshold set and status=10)

### Session Enrollment Status Values

| Value | Description |
|---|---|
| -4 | waitlisted |
| -3 | partially_attended |
| -2 | cancelled |
| -1 | no_show |
| 1 | not_started |
| 2 | in_progress |
| 3 | completed |
| 4 | passed |
| 5 | failed |
| 6 | pending_review |
| 7 | closed |
| 8 | past_due |

---

## Groups

Groups let you invite and enroll multiple users together.

### Methods: Groups

| Method | Description |
|---|---|
| `GET /groups` | List all groups |
| `GET /groups?title=` | Search groups by name |
| `POST /groups` | Create a group |
| `PUT /groups/{id}` | Update a group |
| `DELETE /groups/{id}` | Delete a group |

### Search for Groups

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/groups?title=Sales
```

### Group Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Group identifier |
| `title` | string | Group title |
| `description` | text | Group description |
| `created_at` | date/time | Creation date (UTC) |
| `updated_at` | date/time | Last updated (UTC) |
| `number_of_members` | integer | Users in group |

### Create a Group

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"Group":{"title":"New Title","description":"New description"}}' \
  https://yourdomain.learnupon.com/api/v1/groups
```

| Parameter | Type | Description |
|---|---|---|
| `title` | string | Group title (mandatory, must be unique) |
| `description` | string | Group description (optional) |
| `sso_sync` | boolean | Sync with SAML SSO (optional, default: false) |

### Delete a Group

```bash
curl -X DELETE -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"delete_enrollments":"true"}' \
  https://yourdomain.learnupon.com/api/v1/groups/12345
```

| Parameter | Type | Description |
|---|---|---|
| `delete_enrollments` | boolean | Delete Not Started/In Progress enrollments (optional) |

---

## Portals

The portals resource lets you create, search for, update and delete sub-portals.

### Methods: Portals

| Method | Description |
|---|---|
| `GET /portals` | Search for sub-portals |
| `GET /portals?title=` | Search by name |
| `POST /portals` | Create a sub-portal |
| `GET /portals/{id}/generate_keys` | Generate API keys |
| `PUT /portals/{id}` | Update a sub-portal |
| `GET /portals/membership_types` | Search membership types |
| `DELETE /portals/{id}` | Delete a sub-portal |

### Create a Portal

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"Portal":{"subdomain":"clientx","title":"New Title","description":"New description","header_color":"#419BBD","navigation_color":"#256188"}}' \
  https://yourdomain.learnupon.com/api/v1/portals
```

#### Mandatory Parameters

| Parameter | Type | Description |
|---|---|---|
| `subdomain` | string | Portal subdomain |
| `title` | string | Portal title |

#### Optional Parameters

| Parameter | Type | Description |
|---|---|---|
| `logo_image_url` | string | Logo image URL |
| `store_banner_image_url` | string | Store banner URL |
| `header_color` | string | Header hex color (default: `#419BBD`) |
| `navigation_color` | string | Navigation hex color (default: `#256188`) |
| `number_licenses_purchased` | integer | License count |
| `copy_from_parent` | boolean | Copy settings from top-level portal |
| `allow_course_authoring` | boolean | Allow course editing (default: true) |

### Portal Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Portal identifier |
| `title` | string | Portal title |
| `subdomain` | string | Portal subdomain |
| `created_at` | date/time | Creation date (UTC) |
| `updated_at` | date/time | Last updated (UTC) |
| `allow_course_authoring` | boolean | Course editing allowed |
| `number_of_members` | integer | Users in portal |
| `number_licenses_purchased` | integer | Purchased licenses |
| `number_licenses_used` | integer | Used licenses |

### Generate API Keys

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/portals/12345/generate_keys
```

Returns: `id`, `username`, `password`

---

## Group Memberships

Assign users to groups or remove them. Users can belong to multiple groups.

### Methods: GroupMemberships

| Method | Description |
|---|---|
| `GET /group_memberships?group_id={id}` | Users in a group |
| `GET /group_memberships?group_id={id}&version_id=1.1` | Users with membership ID |
| `GET /group_memberships?user_id={id}` | User's group memberships |
| `POST /group_memberships` | Assign user to group |
| `DELETE /group_memberships/{groupmembership_id}` | Remove user from group |

### Create a GroupMembership

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"GroupMembership":{"group_id":6789,"user_id":1234}}' \
  https://yourdomain.learnupon.com/api/v1/group_memberships
```

| Parameter | Type | Description |
|---|---|---|
| `group_id` | integer | Group to assign to (mandatory) |
| `user_id` | integer | User to assign (mandatory) |
| `process_enrollments` | boolean | Auto-enroll in group courses (default: true) |

### Delete a GroupMembership

```bash
# By groupmembership_id
curl -X DELETE ... https://yourdomain.learnupon.com/api/v1/group_memberships/1234

# By group_id + user_id (use id=0)
curl -X DELETE -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"GroupMembership":{"group_id":6789,"user_id":1234}}' \
  https://yourdomain.learnupon.com/api/v1/group_memberships/0
```

| Parameter | Type | Description |
|---|---|---|
| `process_unenrollments` | boolean | Unenroll from group courses (default: false) |

---

## Group Invites

Invite multiple users into a portal or group in one API call.

### Methods: GroupInvites

| Method | Description |
|---|---|
| `GET /group_invites` | Search invites on portal |
| `GET /group_invites?group_id={group_id}` | Search by group |
| `GET /group_invites/{group_invite_id}` | Search by invite ID |
| `POST /group_invites` | Create invites |
| `DELETE /group_invites/{group_invite_id}` | Delete invite |

### Create a GroupInvite

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"GroupInvite":{"email_addresses":"someone@samplelearningco.com","group_id":6789,"group_membership_type_id":1}}' \
  https://yourdomain.learnupon.com/api/v1/group_invites
```

| Parameter | Type | Description |
|---|---|---|
| `email_addresses` | string | Comma-separated emails (mandatory) |
| `group_id` | integer | Group to invite to (optional) |
| `group_membership_type_id` | integer | 1=learner, 2=admin, 3=instructor, 4=manager |

### GroupInvite Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Invite identifier |
| `invite_status` | string | One of: `sent`, `accepted`, `rejected`, `canceled` |
| `invite_email_address` | string | Destination email |
| `accept_url` | string | URL for accepting invite |
| `group_id` | integer | Associated group |

---

## Group Invite Course Enrollment

Invite users and enroll them onto courses simultaneously. Creates a pending enrollment that activates when the user accepts the invite.

### Methods

| Method | Description |
|---|---|
| `GET /group_invite_course_enrollments` | List all |
| `GET /group_invite_course_enrollments/{id}` | Search by ID |
| `GET /group_invite_course_enrollments?course_id={course_id}` | Search by course |
| `GET /group_invite_course_enrollments?group_id={group_id}` | Search by group |
| `POST /group_invite_course_enrollments` | Create |
| `DELETE /group_invite_course_enrollments/{id}` | Delete |

### Create

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"GroupInviteCourseEnrollment":{"group_invite_id":12345,"course_id":6789}}' \
  https://yourdomain.learnupon.com/api/v1/group_invite_course_enrollments
```

| Parameter | Type | Description |
|---|---|---|
| `group_invite_id` | integer | GroupInvite identifier (mandatory) |
| `course_id` | integer | Course identifier |
| `course_name` | string | Course title (alternative to course_id) |

---

## Group Courses

Associate a course with a group to auto-enroll group members.

### Methods: GroupCourses

| Method | Description |
|---|---|
| `GET /group_courses` | List all GroupCourses |
| `GET /group_courses?group_id={group_id}` | Courses for a group |
| `GET /group_courses?course_id={course_id}` | Groups for a course |
| `POST /group_courses` | Create a GroupCourse |
| `DELETE /group_courses/{group_courses_id}` | Delete a GroupCourse |

### Create a GroupCourse

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"GroupCourse":{"group_id":"5127","course_id":"12360"},"re_enroll_completed":"true"}' \
  https://yourdomain.learnupon.com/api/v1/group_courses
```

| Parameter | Type | Description |
|---|---|---|
| `group_id` | integer | Group identifier (mandatory) |
| `course_id` | integer | Course identifier (mandatory) |
| `re_enroll_completed` | boolean | Re-enroll completed users (mandatory) |

### Delete a GroupCourse

| Parameter | Type | Description |
|---|---|---|
| `id` | integer | GroupCourse identifier (mandatory) |
| `delete_enrollments` | boolean | Delete incomplete enrollments (default: false) |

---

## Group Managers

Connect a manager user with a group.

### Methods: GroupManagers

| Method | Description |
|---|---|
| `GET /group_managers` | List all GroupManagers |
| `GET /group_managers?group_id={group_id}` | Search by group |
| `GET /group_managers?user_id={user_id}` | Search by user |
| `GET /group_managers?email={email}` | Search by email |
| `POST /group_managers` | Create GroupManager |
| `DELETE /group_managers/{id}` | Delete GroupManager |

### Create GroupManager

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"GroupManager":{"group_id":"5127","user_id":"12360","can_create_users":true}}' \
  https://yourdomain.learnupon.com/api/v1/group_managers
```

| Parameter | Type | Description |
|---|---|---|
| `group_id` | integer | Group identifier (mandatory) |
| `user_id` | integer | User identifier (mandatory) |
| `can_create_users` | boolean | Can create users in group (default: false) |

---

## Course Instructors

Associate an instructor user with a course.

### Methods: CourseInstructor

| Method | Description |
|---|---|
| `GET /course_instructors` | List all |
| `GET /course_instructors?course_id={course_id}` | Search by course |
| `GET /course_instructors?user_id={user_id}` | Search by user |
| `GET /course_instructors?email={email}` | Search by email |
| `POST /course_instructors` | Create |
| `DELETE /course_instructors/{id}` | Delete |

### Create a CourseInstructor

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"CourseInstructor":{"course_id":"5127","user_id":"12360"}}' \
  https://yourdomain.learnupon.com/api/v1/course_instructors
```

---

## Learning Paths

A learning path is a customized collection of courses (curriculum).

### Methods: Learning Paths

| Method | Description |
|---|---|
| `GET /learning_paths` | Search for learning paths |
| `POST /learning_paths/{id}/assign_certificate` | Add certificate to path |

### Search for Learning Paths

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/learning_paths
```

#### Optional Parameters

| Parameter | Type | Description |
|---|---|---|
| `name` | string | Learning path name (pattern matching) |
| `id` | integer | Learning path identifier |
| `include_draft` | boolean | Include drafts (default: false) |
| `include_courses` | boolean | Include course list (default: false) |

### Learning Path Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Learning path identifier |
| `name` | string | Title |
| `created_at` | date/time | Creation date (UTC) |
| `sellable` | boolean | Published to store (default: false) |
| `cataloged` | boolean | Published to catalog (default: false) |
| `date_published` | date/time | Publish date (UTC) |
| `keywords` | string | Search keywords |
| `course_length` | float | Length |
| `path_length_unit` | string | `hours` or `minutes` |
| `price` | integer | Price in cents |
| `published_status_id` | string | One of: `published`, `draft`, `archived` |
| `difficulty_level` | string | One of: `not_applicable`, `basic`, `intermediate`, `advanced` |
| `description_html` | string | Description with HTML |
| `description_text` | string | Description raw text |
| `credits_to_be_awarded` | string | Credits list |
| `due_days_after_enrollment` | integer | Days until due |
| `thumbnail_image_url` | string | Thumbnail URL |
| `courses` | array | Course list (if `include_courses=true`) |

---

## Learning Path Enrollments

### Methods

| Method | Description |
|---|---|
| `GET /learning_path_enrollments/{id}` | Search by enrollment id |
| `GET /learning_path_enrollments/search?user_id=` | Search by user |
| `GET /learning_path_enrollments/search?email=` | Search by email |
| `GET /learning_path_enrollments/search?username=` | Search by username |
| `GET /learning_path_enrollments/search?lp_id=` | Search by learning path ID |
| `GET /learning_path_enrollments/search?lp_name=` | Search by learning path name |
| `POST /learning_path_enrollments` | Create enrollment |

### Create Learning Path Enrollment

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"Enrollment":{"email":"learnuponapi@samplelearningco.com","lp_name":"Hello API"}}' \
  https://yourdomain.learnupon.com/api/v1/learning_path_enrollments
```

#### Mandatory Parameters

| Parameter | Type | Description |
|---|---|---|
| `user_id` / `email` / `username` | varies | User identifier (one required) |
| `lp_name` / `lp_id` | varies | Learning path identifier (one required) |

#### Optional Parameters

| Parameter | Type | Description |
|---|---|---|
| `re_enroll_if_completed` | boolean | Re-enroll if completed (default: false) |

### Enrollment Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Enrollment identifier |
| `percentage` | integer | Score achieved |
| `date_started` | date/time | Start date (UTC) |
| `date_completed` | date/time | Completion date (UTC) |
| `date_enrolled` | date/time | Enrollment date (UTC) |
| `lp_id` | integer | Learning path identifier |
| `lp_name` | string | Learning path name |
| `user_id` | integer | User identifier |
| `status` | string | One of: `not_started`, `in_progress`, `completed` |
| `certificate_name` | string | Certificate name |
| `cert_expires_at` | date/time | Certificate expiration |
| `was_recertified` | boolean | Recertification applied |

---

## Gamification

### Methods: Badges

| Method | Description |
|---|---|
| `GET /badges` | List all badges |
| `GET /badges?version_id=1.1` | List badges with image URL |

### Badge Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | integer | Badge identifier |
| `name` | string | Badge name |
| `points` | integer | Points awarded |
| `is_used` | boolean | Badge in use (default: true) |
| `badge_type_id` | integer | Badge type identifier |
| `badge_type` | string | One of: `level_badge`, `learning_badge`, `activity_badge` |
| `image_url` | string | Badge image URL |

### Methods: Leaderboards

| Method | Description |
|---|---|
| `GET /leaderboard` | List leaderboard data |
| `GET /leaderboard?user_id=` | Search by user |
| `GET /leaderboard?group_id=` | Search by group |

#### Optional Parameters

| Parameter | Type | Description |
|---|---|---|
| `group_id` | integer | Filter by group |
| `user_id` | integer | Filter by user |
| `points_over` | integer | Minimum points |
| `points_under` | integer | Maximum points |
| `date_from` | date/time | Start date |
| `date_to` | date/time | End date |
| `level_name` | string | Filter by level |

### Leaderboard Attributes

| Attribute | Type | Description |
|---|---|---|
| `user_id` | integer | User identifier |
| `login` | string | Username or email |
| `name` | string | Display name |
| `position` | integer | Leaderboard position |
| `level` | string | User's level name |
| `total_points` | integer | Total points |
| `total_badges` | integer | Total badges |

---

## Audit Trails

Search for high-priority actions on the portal within the last 30 days.

### Methods

| Method | Description |
|---|---|
| `GET /audit-trails` | Search by action or date range |

```bash
# By action
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/audit-trails?action=8

# By date range
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/audit-trails?date_range=2024-04-10T23:00:00.000Z,2024-05-07T23:00:00.000Z
```

### Action IDs

| Action # | Action |
|---|---|
| 1 | Set complete |
| 2 | User unenrolled |
| 3 | User deleted |
| 4 | User login disabled |
| 5 | User login enabled |
| 6 | Course deleted |
| 7 | Course archived |
| 8 | Learning path deleted |
| 9 | Learning path archived |
| 10 | Exam attempts reset |
| 11 | Group deleted |
| 12 | User removed from a group |

### Audit Trail Attributes

| Attribute | Type | Description |
|---|---|---|
| `action` | integer | Action type ID |
| `initiated_by_user` | string | Person who started the action |
| `initiated_by_user_email` | string | Their email |
| `impacted_user` | string | Person affected |
| `impacted_user_email` | string | Their email |
| `impacted_data` | string | Content/group/person affected |
| `date_and_time_utc` | date/time | Timestamp (UTC) |

---

## Certifications

List certificates available on a portal to use with courses and learning paths.

### Methods

| Method | Description |
|---|---|
| `GET /certifications` | List all certificates |

```bash
curl -X GET -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c \
  https://yourdomain.learnupon.com/api/v1/certifications
```

### Attributes

| Attribute | Type | Description |
|---|---|---|
| `guid` | string | Unique alphanumeric identifier |
| `title` | string | Certificate name |

---

## Batch User Upload via SFTP

Upload CSV files to create/invite/update users in bulk.

### Methods

| Method | Description |
|---|---|
| `POST /batch/upload` | Upload CSV to create/invite users |
| `POST /batch/upload_and_sync` | Upload and sync (differential updates) |

```bash
# Basic upload
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"user_id":"12345","sftp_username":"example_username","sftp_password":"example_password","file_url":"sftp://198.51.100.1:22/example_file.csv","batch_params":{"invite_users":false,"update_users":false,"group_sync":false}}' \
  https://yourdomain.learnupon.com/api/v1/batch/upload
```

### Mandatory Parameters

| Parameter | Type | Description |
|---|---|---|
| `user_id` | string | LearnUpon admin user ID (receives summary email) |
| `sftp_username` | string | SFTP server username |
| `sftp_password` | string | SFTP server password |
| `file_url` | string | SFTP URL (format: `sftp://ip:port/filename.csv`) |

### batch_params Object

| Attribute | Type | Description |
|---|---|---|
| `invite_users` | boolean | Invite instead of create (default: false) |
| `update_users` | boolean | Update existing users (default: false) |
| `group_sync` | boolean | Full group sync (default: false) |
| `process_unenrollments` | boolean | Unenroll when removed from groups (default: false, requires `group_sync=true`) |

### Prerequisites

- LearnUpon must add your SFTP server IP to the allow list
- SFTP versions 3+
- Max file size: 50MB
- Only one batch import at a time

---

## Bulk Operations

Submit large batches of enrollments or unenrollments for asynchronous processing.

### Methods

| Method | Description |
|---|---|
| `POST /bulk-operations/submit/{operation}` | Submit a bulk job |
| `GET /bulk-operations/jobs/{job_id}` | Get job status |
| `GET /bulk-operations/jobs` | List jobs |

### Submit a Bulk Operation

```bash
# Bulk enrollment
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"items":[{"course_id":1321,"user_id":555},{"course_id":1321,"user_id":556}]}' \
  https://yourdomain.learnupon.com/api/v1/bulk-operations/submit/bulk-enrollment

# Bulk unenrollment
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"items":[{"enrollment_id":9901},{"enrollment_id":9902}]}' \
  https://yourdomain.learnupon.com/api/v1/bulk-operations/submit/bulk-unenrollment
```

### Limits

- Up to **100,000 items** per request
- Rate limits apply for concurrent active jobs

### Bulk Enrollment Item Parameters

| Parameter | Type | Description |
|---|---|---|
| `course_id` | integer | Course identifier |
| `user_id` | integer | User identifier |

### Bulk Unenrollment Item Parameters

| Parameter | Type | Description |
|---|---|---|
| `enrollment_id` | integer | Enrollment identifier |

### Job Status Attributes

| Attribute | Type | Description |
|---|---|---|
| `job_id` | string (UUID) | Job identifier |
| `status` | string | One of: `in_progress`, `completed`, `failed`, `aborted`, `timeout` |
| `operation` | string | `bulk-enrollment` or `bulk-unenrollment` |
| `num_operations` | integer | Total items submitted |
| `num_completed_operations` | integer | Successful items |
| `num_failed_operations` | integer | Failed items |
| `created_at` | date/time | Submission time (UTC) |
| `completed_at` | date/time | Completion time (UTC, null if running) |
| `report_url` | string | Results report URL (null until complete) |

### Error Responses

| HTTP Status | Error Code | Description |
|---|---|---|
| 400 | `validation_failed` | Missing or invalid fields |
| 403 | `feature_not_enabled` | Not enabled for portal |
| 404 | `not_found` | Job not found |
| 422 | `payload_too_large` | Exceeds 100,000 items |
| 429 | `rate_limit_exceeded` | Too many concurrent jobs |
| 500 | `internal_error` | Unexpected error |

---

## eCommerce

LearnUpon uses Spree Vendo for eCommerce. The API uses prefixed IDs (e.g., `prod_abcde`, `or_abcde`, `li_abcde`).

### Methods: Products

| Method | Description |
|---|---|
| `GET /ecommerce/products` | Search products |
| `GET /ecommerce/products/{product_id}` | Get product by ID |
| `PATCH /ecommerce/products/{product_id}` | Update product |

### Methods: Prices

| Method | Description |
|---|---|
| `GET /ecommerce/products/{product_id}/prices/{price_type}` | List prices |
| `GET /ecommerce/products/{product_id}/prices/{price_type}/{price_id}` | Get price by ID |
| `POST /ecommerce/products/{product_id}/prices/{price_type}` | Create base price |
| `PATCH /ecommerce/products/{product_id}/prices/{price_type}/{price_id}` | Update price |
| `DELETE /ecommerce/products/{product_id}/prices/{price_type}/{price_id}` | Delete base price |

### Methods: Orders

| Method | Description |
|---|---|
| `GET /ecommerce/orders` | List orders |
| `POST /ecommerce/orders` | Create order |
| `PATCH /ecommerce/orders/{number}` | Update order |
| `PATCH /ecommerce/orders/{number}/advance` | Progress order state |

### Methods: Payments

| Method | Description |
|---|---|
| `GET /ecommerce/payments/{payment_id}` | Get payment |
| `POST /ecommerce/payments` | Create payment |
| `PATCH /ecommerce/payments/{payment_id}` | Update payment |
| `PATCH /ecommerce/payments/{payment_id}/capture` | Capture payment |
| `PATCH /ecommerce/payments/{payment_id}/void` | Void payment |

### Methods: Adjustments

| Method | Description |
|---|---|
| `GET /ecommerce/orders/{order_id}/adjustments` | List adjustments |
| `GET /ecommerce/orders/{order_id}/adjustments/{id}` | Get adjustment |
| `POST /ecommerce/orders/{order_id}/adjustments` | Create adjustment |
| `PATCH /ecommerce/orders/{order_id}/adjustments/{id}` | Update adjustment |
| `DELETE /ecommerce/orders/{order_id}/adjustments/{id}` | Delete adjustment |

### Methods: Line Items

| Method | Description |
|---|---|
| `GET /ecommerce/orders/{order_id}/line_items` | List line items |
| `GET /ecommerce/orders/{order_id}/line_items/{id}` | Get line item |
| `POST /ecommerce/orders/{order_id}/line_items` | Create line item |
| `PATCH /ecommerce/orders/{order_id}/line_items/{id}` | Update line item |
| `DELETE /ecommerce/orders/{order_id}/line_items/{id}` | Delete line item |

### Methods: Other

| Method | Description |
|---|---|
| `GET /ecommerce/customers/{user_id}` | Get customer profile |
| `GET /ecommerce/store_credits/{store_credit_id}` | Get store credit |
| `GET /ecommerce/gift_cards/{gift_card_id}` | Get gift card |

### Product Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | string | Product ID (eCommerce platform) |
| `name` | string | Product title |
| `description` | string | Description (can include HTML) |
| `available_on` | date/time | Available date |
| `discontinue_on` | date/time | Discontinue date |
| `slug` | string | URL text |
| `status` | string | `draft` or `active` |
| `content_type` | string | `course` or `bundle` |
| `content_id` | integer | Portal content identifier (e.g., course_id) |
| `tags` | array | Product tags |
| `base_prices` | array | Prices by currency |
| `membership_prices` | array | Membership prices |
| `tax_category` | object | Tax category data |
| `taxons` | array | Product categories |

### Order Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | string | Order ID |
| `number` | string | Order number (shown to buyers) |
| `state` | string | One of: `cart`, `address`, `delivery`, `payment`, `confirm`, `complete`, `canceled` |
| `payment_state` | string | Payment status |
| `item_total` | string | Sum of line items |
| `total` | string | Total (items + adjustments) |
| `currency` | string | ISO currency code |
| `item_count` | string | Number of items |
| `user_id` | string | Buyer identifier |
| `bill_address` | object | Billing address |
| `line_items` | array | Line item details |
| `payments` | array | Payment details |
| `adjustments` | array | Adjustment details |

### Create an Order

```bash
curl -X POST -H "Content-Type: application/json" \
  --user 988d4f1313f881e5ac6bfdfc7f54244aab:905a12r3a0c -d \
  '{"user_id":29100,"currency":"EUR","channel":"online","considered_risky":false,"line_items_attributes":[{"product_id":"prod_gXYEHJdmfr","quantity":3,"bulk_purchase":true},{"product_id":"prod_CP5SrJgpzQ","quantity":1,"bulk_purchase":false}]}' \
  https://yourdomain.learnupon.com/api/v1/ecommerce/orders
```

### Payment Attributes

| Attribute | Type | Description |
|---|---|---|
| `id` | string | Payment identifier |
| `amount` | string | Payment amount |
| `state` | string | One of: `checkout`, `pending`, `processing`, `balance_due`, `paid`, `completed`, `credit_owed`, `failed`, `void` |
| `payment_method_id` | integer | Payment method identifier |
| `order_id` | string | Associated order |

---

## Appendix: Language Codes

| Value | Description |
|---|---|
| `en` | US English (Default) |
| `cs` | Czech |
| `da` | Danish |
| `en_gb` | UK English |
| `nl` | Dutch |
| `fi_fl` | Finnish |
| `nl_be` | Flemish |
| `fr` | French |
| `fr_ca` | French (Canadian) |
| `de` | German |
| `hu` | Hungarian |
| `id` | Indonesian |
| `it` | Italian |
| `ja` | Japanese |
| `ko` | Korean |
| `nb_no` | Norwegian |
| `pl` | Polish |
| `pt` | Portuguese |
| `pt_br` | Portuguese (Brazilian) |
| `ru` | Russian |
| `sk` | Slovak |
| `es` | Spanish |
| `sv` | Swedish |
| `tr` | Turkish |
| `vi_vn` | Vietnamese |
| `zh_cn` | Chinese (Simplified) |
| `zh_tw` | Chinese (Traditional) |

---

## Appendix: Timezone Names

The following timezones are supported in API calls and in the application. Use the `id` value for API parameters like `timezone_id`.

| ID | Name |
|---|---|
| `International Date Line West` | (GMT-12:00) International Date Line West |
| `American Samoa` | (GMT-11:00) American Samoa |
| `Midway Island` | (GMT-11:00) Midway Island |
| `Hawaii` | (GMT-10:00) Hawaii |
| `Alaska` | (GMT-08:00) Alaska |
| `Arizona` | (GMT-07:00) Arizona |
| `Mazatlan` | (GMT-07:00) Mazatlan |
| `Pacific Time (US & Canada)` | (GMT-07:00) Pacific Time (US & Canada) |
| `Tijuana` | (GMT-07:00) Tijuana |
| `Central America` | (GMT-06:00) Central America |
| `Chihuahua` | (GMT-06:00) Chihuahua |
| `Guadalajara` | (GMT-06:00) Guadalajara |
| `Mexico City` | (GMT-06:00) Mexico City |
| `Monterrey` | (GMT-06:00) Monterrey |
| `Mountain Time (US & Canada)` | (GMT-06:00) Mountain Time (US & Canada) |
| `Saskatchewan` | (GMT-06:00) Saskatchewan |
| `Bogota` | (GMT-05:00) Bogota |
| `Central Time (US & Canada)` | (GMT-05:00) Central Time (US & Canada) |
| `Lima` | (GMT-05:00) Lima |
| `Quito` | (GMT-05:00) Quito |
| `Caracas` | (GMT-04:00) Caracas |
| `Eastern Time (US & Canada)` | (GMT-04:00) Eastern Time (US & Canada) |
| `Georgetown` | (GMT-04:00) Georgetown |
| `Indiana (East)` | (GMT-04:00) Indiana (East) |
| `La Paz` | (GMT-04:00) La Paz |
| `Puerto Rico` | (GMT-04:00) Puerto Rico |
| `Santiago` | (GMT-04:00) Santiago |
| `Atlantic Time (Canada)` | (GMT-03:00) Atlantic Time (Canada) |
| `Brasilia` | (GMT-03:00) Brasilia |
| `Buenos Aires` | (GMT-03:00) Buenos Aires |
| `Montevideo` | (GMT-03:00) Montevideo |
| `Newfoundland` | (GMT-02:30) Newfoundland |
| `Mid-Atlantic` | (GMT-02:00) Mid-Atlantic |
| `Cape Verde Is.` | (GMT-01:00) Cape Verde Is. |
| `Azores` | (GMT+00:00) Azores |
| `Monrovia` | (GMT+00:00) Monrovia |
| `UTC` | (GMT+00:00) UTC |
| `Casablanca` | (GMT+01:00) Casablanca |
| `Dublin` | (GMT+01:00) Dublin |
| `Edinburgh` | (GMT+01:00) Edinburgh |
| `Lisbon` | (GMT+01:00) Lisbon |
| `London` | (GMT+01:00) London |
| `West Central Africa` | (GMT+01:00) West Central Africa |
| `Amsterdam` | (GMT+02:00) Amsterdam |
| `Belgrade` | (GMT+02:00) Belgrade |
| `Berlin` | (GMT+02:00) Berlin |
| `Bern` | (GMT+02:00) Bern |
| `Bratislava` | (GMT+02:00) Bratislava |
| `Brussels` | (GMT+02:00) Brussels |
| `Budapest` | (GMT+02:00) Budapest |
| `Copenhagen` | (GMT+02:00) Copenhagen |
| `Harare` | (GMT+02:00) Harare |
| `Kaliningrad` | (GMT+02:00) Kaliningrad |
| `Ljubljana` | (GMT+02:00) Ljubljana |
| `Madrid` | (GMT+02:00) Madrid |
| `Paris` | (GMT+02:00) Paris |
| `Prague` | (GMT+02:00) Prague |
| `Pretoria` | (GMT+02:00) Pretoria |
| `Rome` | (GMT+02:00) Rome |
| `Sarajevo` | (GMT+02:00) Sarajevo |
| `Skopje` | (GMT+02:00) Skopje |
| `Stockholm` | (GMT+02:00) Stockholm |
| `Vienna` | (GMT+02:00) Vienna |
| `Warsaw` | (GMT+02:00) Warsaw |
| `Zagreb` | (GMT+02:00) Zagreb |
| `Zurich` | (GMT+02:00) Zurich |
| `Athens` | (GMT+03:00) Athens |
| `Baghdad` | (GMT+03:00) Baghdad |
| `Bucharest` | (GMT+03:00) Bucharest |
| `Cairo` | (GMT+03:00) Cairo |
| `Helsinki` | (GMT+03:00) Helsinki |
| `Istanbul` | (GMT+03:00) Istanbul |
| `Jerusalem` | (GMT+03:00) Jerusalem |
| `Kuwait` | (GMT+03:00) Kuwait |
| `Minsk` | (GMT+03:00) Minsk |
| `Moscow` | (GMT+03:00) Moscow |
| `Nairobi` | (GMT+03:00) Nairobi |
| `Riga` | (GMT+03:00) Riga |
| `Riyadh` | (GMT+03:00) Riyadh |
| `Sofia` | (GMT+03:00) Sofia |
| `St. Petersburg` | (GMT+03:00) St. Petersburg |
| `Tallinn` | (GMT+03:00) Tallinn |
| `Vilnius` | (GMT+03:00) Vilnius |
| `Volgograd` | (GMT+03:00) Volgograd |
| `Tehran` | (GMT+03:30) Tehran |
| `Abu Dhabi` | (GMT+04:00) Abu Dhabi |
| `Baku` | (GMT+04:00) Baku |
| `Muscat` | (GMT+04:00) Muscat |
| `Samara` | (GMT+04:00) Samara |
| `Tbilisi` | (GMT+04:00) Tbilisi |
| `Yerevan` | (GMT+04:00) Yerevan |
| `Kabul` | (GMT+04:30) Kabul |
| `Almaty` | (GMT+05:00) Almaty |
| `Astana` | (GMT+05:00) Astana |
| `Ekaterinburg` | (GMT+05:00) Ekaterinburg |
| `Islamabad` | (GMT+05:00) Islamabad |
| `Karachi` | (GMT+05:00) Karachi |
| `Tashkent` | (GMT+05:00) Tashkent |
| `Chennai` | (GMT+05:30) Chennai |
| `Kolkata` | (GMT+05:30) Kolkata |
| `Mumbai` | (GMT+05:30) Mumbai |
| `New Delhi` | (GMT+05:30) New Delhi |
| `Sri Jayawardenepura` | (GMT+05:30) Sri Jayawardenepura |
| `Kathmandu` | (GMT+05:45) Kathmandu |
| `Dhaka` | (GMT+06:00) Dhaka |
| `Urumqi` | (GMT+06:00) Urumqi |
| `Bangkok` | (GMT+07:00) Bangkok |
| `Hanoi` | (GMT+07:00) Hanoi |
| `Jakarta` | (GMT+07:00) Jakarta |
| `Krasnoyarsk` | (GMT+07:00) Krasnoyarsk |
| `Novosibirsk` | (GMT+07:00) Novosibirsk |
| `Beijing` | (GMT+08:00) Beijing |
| `Chongqing` | (GMT+08:00) Chongqing |
| `Hong Kong` | (GMT+08:00) Hong Kong |
| `Irkutsk` | (GMT+08:00) Irkutsk |
| `Kuala Lumpur` | (GMT+08:00) Kuala Lumpur |
| `Perth` | (GMT+08:00) Perth |
| `Singapore` | (GMT+08:00) Singapore |
| `Taipei` | (GMT+08:00) Taipei |
| `Ulaanbaatar` | (GMT+08:00) Ulaanbaatar |
| `Osaka` | (GMT+09:00) Osaka |
| `Sapporo` | (GMT+09:00) Sapporo |
| `Seoul` | (GMT+09:00) Seoul |
| `Tokyo` | (GMT+09:00) Tokyo |
| `Yakutsk` | (GMT+09:00) Yakutsk |
| `Adelaide` | (GMT+09:30) Adelaide |
| `Darwin` | (GMT+09:30) Darwin |
| `Brisbane` | (GMT+10:00) Brisbane |
| `Canberra` | (GMT+10:00) Canberra |
| `Guam` | (GMT+10:00) Guam |
| `Hobart` | (GMT+10:00) Hobart |
| `Melbourne` | (GMT+10:00) Melbourne |
| `Port Moresby` | (GMT+10:00) Port Moresby |
| `Sydney` | (GMT+10:00) Sydney |
| `Vladivostok` | (GMT+10:00) Vladivostok |
| `Magadan` | (GMT+11:00) Magadan |
| `New Caledonia` | (GMT+11:00) New Caledonia |
| `Solomon Is.` | (GMT+11:00) Solomon Is. |
| `Srednekolymsk` | (GMT+11:00) Srednekolymsk |
| `Auckland` | (GMT+12:00) Auckland |
| `Fiji` | (GMT+12:00) Fiji |
| `Kamchatka` | (GMT+12:00) Kamchatka |
| `Marshall Is.` | (GMT+12:00) Marshall Is. |
| `Wellington` | (GMT+12:00) Wellington |
| `Chatham Is.` | (GMT+12:45) Chatham Is. |
| `Nuku'alofa` | (GMT+13:00) Nuku'alofa |
| `Samoa` | (GMT+13:00) Samoa |
| `Tokelau Is.` | (GMT+13:00) Tokelau Is. |
