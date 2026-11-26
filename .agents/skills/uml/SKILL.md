---
name: uml
description: Write or refresh the PlantUML use case diagram (uml/use-case.wsd), class diagram of the model classes (uml/class.wsd) and one story's sequence diagram (uml/sequence-US#.wsd), render them and embed them in the README. Due by the end of Sprint 1, and re-run whenever the model or the flow changes.
---

# /uml

Three diagrams, each **+5** in the Documentation rubric: use case, sequence, class. All three must be **in the README**, not only in `uml/`. Take which one from the request; with none, do all three.

## Sources

- **Use case diagram**: the user stories in [README.md](../../../README.md#user-stories). Every use case maps to a story; every story appears.
- **Class diagram**: `src/app/models.py` once it exists - read the code, a diagram drawn from memory drifts. Before the code exists, the acceptance criteria and decisions from `/new-design`.
- **Sequence diagram**: the story's route, service and repository code, and its criteria.

## Use case diagram

`uml/use-case.wsd` already exists. Actors: Visitor, Member, Project owner (a specialisation of Member). US1-US11 all appear. Keep the `(US#)` in each use case name so the grader can trace it, and update it if a story is added or dropped.

## Class diagram

`uml/class.wsd` - **model classes only**. The assignment says no framework, library, controller, route, repository, service, test or utility classes. Show attributes, key methods, and associations **with multiplicities**. The template ships a stub `User` - replace it.

```plantuml
@startuml class
hide circle
skinparam classAttributeIconSize 0

enum ProjectRole {
  OWNER
  MEMBER
}

enum IssueType {
  STORY
  BUG
  TASK
}

enum Priority {
  LOW
  MEDIUM
  HIGH
}

enum IssueStatus {
  TO_DO
  IN_PROGRESS
  DONE
}

enum SprintStatus {
  PLANNED
  ACTIVE
  COMPLETED
}

class User {
  +id : Integer <<PK>>
  +username : String <<unique>>
  +email : String <<unique>>
  +password_hash : String
  +created_at : DateTime
  +set_password(password)
  +check_password(password) : bool
}

class Project {
  +id : Integer <<PK>>
  +name : String
  +key : String(6) <<unique>>
  +next_issue_number : Integer
  +created_at : DateTime
}

class ProjectMember {
  +user_id : Integer <<FK>>
  +project_id : Integer <<FK>>
  +role : ProjectRole
}

class Issue {
  +id : Integer <<PK>>
  +project_id : Integer <<FK>>
  +number : Integer
  +title : String
  +description : Text
  +type : IssueType
  +priority : Priority
  +status : IssueStatus
  +story_points : Integer {1,2,3,5,8,13}
  +assignee_id : Integer <<FK>> [0..1]
  +sprint_id : Integer <<FK>> [0..1]
  +key() : String
}

class Sprint {
  +id : Integer <<PK>>
  +project_id : Integer <<FK>>
  +name : String
  +start_date : Date
  +end_date : Date
  +status : SprintStatus
  +total_points() : Integer
}

User "1" -- "0..*" ProjectMember
Project "1" *-- "1..*" ProjectMember : members >
Project "1" *-- "0..*" Issue : contains >
Project "1" *-- "0..*" Sprint : plans >
Sprint "0..1" o-- "0..*" Issue : includes >
User "0..1" -- "0..*" Issue : assigned to <
ProjectMember ..> ProjectRole
Issue ..> IssueType
Issue ..> Priority
Issue ..> IssueStatus
Sprint ..> SprintStatus

note "An issue with no sprint is in the backlog.\nAt most one ACTIVE sprint per project." as N1
@enduml
```

Add `Comment` (`id`, `issue_id`, `author_id`, `body`, `created_at`; `Issue "1" *-- "0..*" Comment`, `User "1" -- "0..*" Comment : writes >`) - the foundation creates it with the other models. This is a starting shape, not the answer - match it to the actual models.

## Sequence diagram

`uml/sequence-US<n>.wsd` (the template `uml/sequence-US#.wsd` is a placeholder - replace it) shows **one implemented story's flow, not registration or login**, from the browser through **all three layers** - route, service, repository - to PostgreSQL, with **at least one alternative path**. US7 (start a sprint) is the suggested choice because its rule gives a natural `alt`.

```plantuml
@startuml sequence-US7
actor Owner
participant Browser
participant "sprints blueprint\nstart_sprint()" as Route
participant "SprintService" as Service
participant "SprintRepository" as Repo
database PostgreSQL as DB

Owner -> Browser : click Start sprint
Browser -> Route : POST /projects/<key>/sprints/<id>/start
Route -> Service : start_sprint(user, sprint_id)
Service -> Repo : get_membership(user, project)
Repo -> DB : SELECT project_member
alt not the owner
  Service --> Route : raise PermissionDenied
  Route --> Browser : 403
else owner
  Service -> Repo : find_active(project_id)
  Repo -> DB : SELECT sprint WHERE status = ACTIVE
  alt another sprint is active
    Service --> Route : raise ValidationError
    Route --> Browser : 200, "Another sprint is already active"
  else none active
    Service -> Repo : set_status(sprint, ACTIVE)
    Repo -> DB : UPDATE sprint; COMMIT
    Service --> Route : sprint
    Route --> Browser : 302 -> board
    Browser --> Owner : board shows the sprint's issues
  end
end
@enduml
```

Every message is something the code actually does, in the order it does it.

## Render and embed

PlantUML is not installed by default. Either the PlantUML extension in VS Code / IntelliJ (export PNG), or:

```
java -jar plantuml.jar uml/use-case.wsd uml/class.wsd uml/sequence-US7.wsd
```

Commit the PNGs beside the sources and embed each in its README section (`![Class diagram](uml/class.png)`), so the grader does not have to render them.

## Keep them true

A diagram that disagrees with the code is worth less than one that is plainly incomplete. Re-run after any `/new-model`, and when the story's flow changes.
