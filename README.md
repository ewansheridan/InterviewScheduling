# Interview Scheduling

A MySQL relational database for managing recruitment data (candidates, departments, job positions, skills, interviews, job offers) to schedule job interviews for many candidates applying for many positions.

This project was originally an assignment for the 'Databases and Information Systems' module in UCD.

## How to run

#### Requirements

- MySQL 8.0; the dump was exported from MySQL 8.0.39
- The MySQL command-line client or MySQL Workbench (these instructions detail how to use it on the command line specifically)

#### Import the database

1. Clone the repository:

   ```bash
   git clone https://github.com/ewansheridan/InterviewScheduling.git
   cd InterviewScheduling
   ```

2. Start the MySQL client (use your normal username & password):

   ```bash
   mysql -u root -p
   ```

3. Create a database, select it, and import the SQL file:

   ```sql
   CREATE DATABASE interview_scheduling;
   USE interview_scheduling;
   SOURCE sheridan23336833.sql;
   ```

   Run the client from the repository directory, or supply the full path to the SQL file. The dump does not create or select a database itself. **Make sure to import into a fresh database:** the script drops and recreates tables with the same names, so do not accidentally run it inside one of your own databases.

## Features
- Related tables with primary keys, foreign keys, and composite keys
- Many-to-many relationships
- Unique constraints on candidate telephone numbers and department names
- Interview dates and an offer status constrained to `0` or `1`
  - Essentially had to manually create a Boolean type in SQL
- 7 insertion procedures and 11 reporting procedures (detailed below)
- Included sample data for exploring the schema and queries
  - Sample data already pre-populates the tables when you clone this repo

## Database structure

| Table | Purpose |
| --- | --- |
| `candidates` | Candidate names and contact details |
| `departments` | Department names and contact details |
| `positions` | Job positions and their departments (e.g. an `Accountant` position for the `Finance` department)|
| `skills` | Available skill names (e.g. `manual handling`, `sales`, `ECDL`, etc.)|
| `candidate_skills` | Skills held by each candidate |
| `position_skills_req` | Skills required by each position |
| `interviews` | Interview dates, candidates, positions, and offer outcomes |

_Each department can have multiple positions. Candidates and positions can each have multiple skills through their respective junction tables. Each interview references one candidate and one position._



## Example queries

After importing, select the database:

```sql
USE interview_scheduling;
```

<br>Add a skill:
```sql
CALL add_skill('Software Development');
```

<br>Add a candidate: 
<br>_(save their generated ID to make the other procedures more straightforward)_
```sql
CALL add_candidate('Alex', 'Murphy', '10 Example Road', '0800000000');
SET @candidate_id = LAST_INSERT_ID();
```


<br>Assign the skill to the candidate:
```sql
CALL add_candidate_skill(@candidate_id, 'Software Development');
```

<br>Add a department:
<br>_(save its generated ID to use it in other procedures)_
```sql
CALL add_department('Engineering', '20 Example Street', '087 123 4567');
SET @department_id = LAST_INSERT_ID();
```


<br>Add a position within that department:
```sql
CALL add_position(@department_id, 'Software Engineer');
SET @position_id = LAST_INSERT_ID();
```

<br>Specify a required skill for the position:
```sql
CALL add_req_skill(@position_id, 'Software Development');
```


<br>Record an interview: (`0` = no offer, `1` = offer made):
```sql
CALL add_interview('2026-10-15', @candidate_id, @position_id, 0);
```

    
<br>Find candidates by first name:
```sql
CALL Q1_CandsWithGivenFN('Danielle');
```

<br>Find candidates with at least one required skill for position 310:
```sql
CALL Q4_CandsWith1SkillForPositId(310);
```

<br>List positions requiring a particular skill:
```sql
CALL Q5_PositsReqGivenSkill('Data Analysis');
```

<br>Count distinct candidates who received an offer:
```sql
CALL Q6_NoCandsWithOffers();
```

<br>List interviews scheduled on a specific date:
```sql
CALL Q9_InterviewsFromDate('2025-01-13');
```

<br>Find candidates with more than one interview:
```sql
CALL Q11_NameAndIDofCandsMultipleInterviews();
```


## Scope and implementation notes

This repository contains the database schema, sample records, and stored procedures. Interviews are recorded by date; automatic scheduling, time slots, and conflict detection are not implemented.

The skill-matching query returns candidates with **at least one** required skill, rather than checking that they meet every requirement.
