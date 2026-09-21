# Oracle PDB Assignment II

**Student:** Abdelgafar
**Student ID:** 28874
**Course:** PL/SQL (AUCA)

---

## 1. Overview of Tasks

| Task | Title | Goal |
|------|-------|------|
| 1 | Create a New Pluggable Database | Create the PDB `AB_PDB_28874` and the user `Abdelgafar_plsqlauca_28874` inside it |
| 2 | Create and Delete a PDB | Create the temporary PDB `AB_TO_DELETE_PDB_28874`, verify it, delete it completely, confirm it is gone |
| 3 | Oracle Enterprise Manager (OEM) | Open the EM Express dashboard and show the username and the PDB |
| 4 | Documentation & Reporting | Publish this report and the screenshots in a public GitHub repository |

---

## 2. Oracle Environment Used

| Item | Value |
|------|-------|
| Database | Oracle Database 21c Enterprise Edition (21.3.0.0.0) |
| Operating system | Microsoft Windows x86 64-bit |
| Container database (CDB) | `ORCL` |
| Existing PDBs | `PDB$SEED`, `ORCLPDB` |
| Tools | Oracle SQL Developer (connection as `SYS` with the `SYSDBA` role), Oracle EM Express in the web browser |

---

## 3. Task 1 — Create a New Pluggable Database

### 1.1 — PDB Creation Command

**Requirement:** Create the pluggable database `AB_PDB_28874`.

I connected to the root container (`CDB$ROOT`) as `SYS` with the `SYSDBA` role, looked up the seed datafile location, then created the PDB by copying from `PDB$SEED`.

```sql
SHOW CON_NAME;
SELECT name FROM v$datafile WHERE con_id = 2;

CREATE PLUGGABLE DATABASE AB_PDB_28874
  ADMIN USER pdb_admin IDENTIFIED BY "<password>"
  FILE_NAME_CONVERT = ('C:\APP\ORADATA\ORCL\PDBSEED\',
                       'C:\APP\ORADATA\ORCL\AB_PDB_28874\');
```

![PDB creation command](screenshots/pdb_creation/task1_1_pdb_creation_command.png)

### 1.2 — PDB Open State

**Requirement:** Open the PDB and confirm it is in `READ WRITE` mode.

After creation the PDB is `MOUNTED` by default, so I opened it and saved its state so it reopens automatically after a restart.

```sql
ALTER PLUGGABLE DATABASE AB_PDB_28874 OPEN;
SHOW PDBS;
ALTER PLUGGABLE DATABASE AB_PDB_28874 SAVE STATE;
```

![PDB open state](screenshots/pdb_creation/task1_2_pdb_open_state.png)

### 1.3 — User Created Inside the PDB

**Requirement:** Create the user `Abdelgafar_plsqlauca_28874` inside the PDB, with the username clearly visible.

I switched the session into the PDB with `ALTER SESSION SET CONTAINER`, then created the user and granted it the privileges it needs for future assignments.

```sql
ALTER SESSION SET CONTAINER = AB_PDB_28874;
SHOW CON_NAME;

CREATE USER Abdelgafar_plsqlauca_28874 IDENTIFIED BY "<password>";
GRANT CONNECT, RESOURCE TO Abdelgafar_plsqlauca_28874;
GRANT UNLIMITED TABLESPACE TO Abdelgafar_plsqlauca_28874;

SELECT username FROM dba_users WHERE username LIKE '%PLSQLAUCA%';
```

![User created inside the PDB](screenshots/pdb_creation/task1_3_user_created_inside_pdb.png)

---

## 4. Task 2 — Create and Delete a PDB

### 2.1 — PDB Creation (Command + Result)

**Requirement:** Create the temporary PDB `AB_TO_DELETE_PDB_28874`.

From `CDB$ROOT` I created a second, temporary PDB the same way as Task 1.

```sql
CREATE PLUGGABLE DATABASE AB_to_delete_pdb_28874
  ADMIN USER pdb_admin IDENTIFIED BY "<password>"
  FILE_NAME_CONVERT = ('C:\APP\ORADATA\ORCL\PDBSEED\',
                       'C:\APP\ORADATA\ORCL\AB_TO_DELETE_PDB_28874\');
```

![Temporary PDB creation](screenshots/pdb_creation/task2_1_pdb_creation.png)

### 2.2 — PDB Deletion (Command + Result)

**Requirement:** Delete the temporary PDB completely and confirm it no longer exists.

I dropped the PDB together with its datafiles so nothing was left on disk, then re-ran `SHOW PDBS` to confirm it was gone.

```sql
DROP PLUGGABLE DATABASE AB_to_delete_pdb_28874 INCLUDING DATAFILES;
SHOW PDBS;
```

![Temporary PDB deletion](screenshots/pdb_deletion/task2_3_pdb_deletion.png)

---

## 5. Task 3 — Oracle Enterprise Manager (OEM)

### 3.1 — OEM Dashboard

**Requirement:** Access OEM and show a dashboard that reflects the Oracle environment and completed PDB tasks, with the username visible.

I used Oracle Enterprise Manager Database Express (EM Express). I enabled a dedicated HTTPS port for my PDB, granted my user the `EM_EXPRESS_ALL` role, and logged in from the browser.

```sql
ALTER SESSION SET CONTAINER = AB_PDB_28874;
GRANT EM_EXPRESS_ALL TO Abdelgafar_plsqlauca_28874;
EXEC DBMS_XDB_CONFIG.SETHTTPSPORT(5502);
ALTER SESSION SET CONTAINER = CDB$ROOT;
```

Accessed via: `https://localhost:5502/em`
Logged in as: `Abdelgafar_plsqlauca_28874`
Container: `AB_PDB_28874`

![EM Express dashboard](screenshots/oem_dashboard/task3_1_oem_dashboard.png)

---

## 6. Task 4 — Documentation & Reporting

This repository contains this README and the screenshots for Tasks 1 to 3, organized in the required folder structure:

```
oracle_pdb_ass_II_28874_Abdelgafar/
├── README.md
└── screenshots/
    ├── pdb_creation/
    ├── pdb_deletion/
    └── oem_dashboard/
```

---

## 7. Challenges Faced and How They Were Solved

| Problem | Cause | Solution |
|---------|-------|----------|
| `ORA-01031: insufficient privileges` when opening the PDB | The connection did not have the `SYSDBA` role | Created a new SQL Developer connection as `SYS` with the `SYSDBA` role and service name `ORCL` |
| `ORA-65012: Pluggable database already exists` | The PDB had already been created by an earlier run of the same command | Dropped the existing PDB with `INCLUDING DATAFILES` and created it again once |
| `ORA-65040: operation not allowed from within a pluggable database` | The session was still inside a PDB after `ALTER SESSION SET CONTAINER` | Returned to the root with `ALTER SESSION SET CONTAINER = CDB$ROOT` before creating or dropping PDBs |
| `ORA-01920: user name conflicts with another user or role name` | The user had already been created | Dropped the user with `DROP USER ... CASCADE` and created it again |
| EM Express kept showing a browser sign-in pop-up | The port configured in `CDB$ROOT` serves the root only, and the user lacked the EM Express role | Configured a dedicated HTTPS port inside the PDB with `DBMS_XDB_CONFIG.SETHTTPSPORT(5502)`, granted `EM_EXPRESS_ALL`, and logged in at `https://localhost:5502/em` |

---

## 8. Integrity Statement

I, Abdelgafar (Student ID 28874), confirm that I ran every command in this assignment myself on my own Oracle installation, and that all screenshots in this repository were taken from my own environment. I used an AI assistant (Claude) for explanations and troubleshooting guidance, and I understand the commands I executed.

---

## 9. Submission Details

```
Repository Link: [GitHub URL]
PDB Name Created: AB_PDB_28874
Issues Encountered: Yes
[add the remaining lines of the required block here]
```
