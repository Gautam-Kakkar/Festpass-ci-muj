# FestPass CI

A Jenkins CI pipeline for **FestPass**, a small command-line campus event registration app written in Java.

| | |
|---|---|
| **Author** | Gautam |
| **Class code** | 001 |
| **Challenge code** | 00152 |
| **Job** | `festpass-ci` on agent `student-laptop` |

## About the project

FestPass lets you create events with a seat capacity, book and cancel seats for students, and view rosters and summaries. Data is stored in a plain tab-separated file.

The focus of this project is the **pipeline**, not the app: a Jenkins job that checks out the code, runs the 28 unit tests, builds a JAR, smoke-tests it, archives the artifacts, and records build evidence for every run.

## What I did

- Configured the Jenkins pipeline job from SCM (`git://127.0.0.1:9418/festpass-java`, branch `*/main`, script path `Jenkinsfile`).
- Added a `pollSCM('* * * * *')` trigger to the `Jenkinsfile` so Jenkins builds automatically on every new commit.
- Personalized the `help` output with my alias.
- Diagnosed the failing tests in Jenkins and fixed the cancellation bug in `BookingService.java`: `cancel()` checked that a student had a booking but never removed it, so the seat stayed taken.

## Project layout

```
Jenkinsfile                 Pipeline definition
submission.properties       Class code and challenge code
src/main/java/              App: FestPass (CLI), BookingService, BookingStore
src/test/java/              28 supplied unit tests
ci/Build.java               test / package / verify tasks
ci/SmokeTest.java           Runs the built JAR end to end
ci/BuildEvidence.java       Writes the build evidence CSV
submission/                 Evidence, CSVs and final JAR
ASSIGNMENT.md               Assignment brief
```

## Pipeline stages

1. **Checkout** – checks out the commit and prints Git and Java versions
2. **Submission setup** – reads `submission.properties` and prepares evidence
3. **Test** – runs the 28 tests and publishes the JUnit report
4. **Build JAR** – packages `build/festpass.jar`
5. **Smoke test JAR** – runs the JAR and checks its output
6. **Save artifact** – archives the JAR, build info and smoke test output

A failed Test stage skips the later stages, so no JAR is published from a red build. The evidence CSV is always archived, even on failure.

## Build and run locally

Requires JDK 17 or newer (the code uses text blocks and switch expressions).

```
java ci/Build.java verify        # test + package
java ci/SmokeTest.java           # smoke test the JAR
java -jar build/festpass.jar help
```

### Example

```
java -jar build/festpass.jar add-event E1 "Music Night" 2
java -jar build/festpass.jar book E1 S1
java -jar build/festpass.jar roster E1
java -jar build/festpass.jar cancel E1 S1
java -jar build/festpass.jar summary
```

### Commands

| Command | Description |
|---|---|
| `add-event ID "TITLE" CAPACITY` | Create an event |
| `list` | List events with booked/capacity |
| `book EVENT_ID STUDENT_ID` | Book a seat |
| `cancel EVENT_ID STUDENT_ID` | Cancel a booking |
| `roster EVENT_ID` | Show students booked on an event |
| `summary` | Totals across all events |

Exit codes: `0` success, `2` invalid request, `3` storage failure.

## Evidence

Red and green build evidence, CSVs and the final JAR are in [`submission/`](submission/), with a short write-up in [`submission/EVIDENCE.md`](submission/EVIDENCE.md).
