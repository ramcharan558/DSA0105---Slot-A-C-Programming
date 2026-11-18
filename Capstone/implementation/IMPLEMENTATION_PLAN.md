# Disruption-Resilient Timetable Management System - Implementation Plan

## 1. Environment Verification
- **Compiler**: GCC 13.1.0 (MinGW-w64 x86_64) located at `C:\Qt\Tools\mingw1310_64\bin\g++.exe`
- **Build System**: CMake 4.4.4 & Ninja / MinGW Makefiles
- **GUI Framework**: Qt 6.8.2 MinGW 64-bit located at `C:\Qt\6.8.2\mingw_64`
- **Third Party**: `nlohmann/json` 3.11.3 (single header) & `doctest` 2.4.11 (single header) in `/third_party`

---

## 2. Project Architecture

```
/
├── CMakeLists.txt
├── third_party/
│   ├── json.hpp
│   └── doctest.h
├── core/
│   ├── CMakeLists.txt
│   ├── model/           # Course, Faculty, Room, StudentGroup, Slot, Assignment, Disruption, Config
│   ├── engine/          # Constraint (ABC + subclasses), Generator, RepairEngine, Validator, Scorer
│   ├── io/              # JsonStorage, CsvExporter, HtmlExporter
│   └── util/            # Exceptions, TimeUtils, RandomUtils
├── cli/
│   ├── CMakeLists.txt
│   └── main.cpp         # Interactive console menu application
├── gui/
│   ├── CMakeLists.txt
│   ├── main.cpp
│   ├── mainwindow.hpp/.cpp
│   ├── theme.hpp/.cpp   # Theme palettes (Light, Dark, Ocean, Sunset) + QSS template
│   ├── widgets/         # GaugeWidget, ScoreBarWidget, TimetableGridWidget, StatCard
│   ├── pages/           # Dashboard, Data, Timetable, Disruptions, RepairReport, Validation, Score, Settings
│   ├── workers/         # SolverWorker (QThread)
│   └── resources/       # SVG icons, sample configs
├── tests/
│   ├── CMakeLists.txt
│   └── test_main.cpp    # Doctest test suite covering Generator, Repair, Validator, Stress test
├── data/
│   ├── sample_department.json
│   ├── sample_config.json
│   ├── demo1_meera_absent.json
│   ├── demo2_room_unavailable.json
│   ├── demo3_day_cancelled.json
│   └── demo4_unresolved.json
└── README.md            # Build/run instructions, architecture, C++ viva concepts table
```

---

## 3. Phased Execution Steps

### Phase 1: Environment & Project Foundation
- Verify Qt6, CMake, and MinGW compilers.
- Download/vendor `json.hpp` and `doctest.h`.
- Create root `CMakeLists.txt` and directory skeleton.

### Phase 2: Core Data Model & Constraints
- Implement `Slot` (with operators `<`, `==`, `<<`), `Segment`, `DayTemplate`, `Course`, `Faculty`, `Room`, `StudentGroup`, `Assignment`, `Disruption`, `Config`.
- Implement `Constraint` abstract base class and concrete constraints:
  - `NoClashConstraint` (faculty, room, group overlap check)
  - `RoomTypeConstraint` (Lab vs Classroom)
  - `RoomCapacityConstraint`
  - `FacultyAvailabilityAndLoadConstraint`
  - `BreakSegmentConstraint`
  - `CourseQuotaConstraint`
  - `DailySubjectLimitConstraint`
- Implement JSON serialization/deserialization for all entities with `nlohmann/json`.

### Phase 3: Core Generator & Independent Validator
- Implement `TimetableGenerator`:
  - MRV (Minimum Remaining Values) variable ordering with degree tie-breaker.
  - Forward checking domain filtering.
  - Configurable node limit, time limit, and random seed.
  - Unsatisfiable diagnostics.
- Implement `IndependentValidator`:
  - Validates hard constraints from scratch without relying on generator internal state.
  - Returns detailed violations (severity, entity, slot, reason).

### Phase 4: Disruption & Repair Engine, Scorer, IO & Unit Tests
- Implement `DisruptionEngine`:
  - Detect broken assignments.
  - Hierarchical minimal-change repair:
    1. Same slot, alternative room.
    2. Nearby slot, same day.
    3. Nearest different day.
    4. 1-level displacement of unpinned, unaffected class.
    5. Mark unresolved if infeasible.
  - Compute weighted cost and produce detailed change log.
- Implement `TimetableScorer`:
  - Computes 0-100 quality score and breakdown (free gaps, repeated subjects, faculty balance, workload distribution).
  - Computes stability percentage.
- Implement HTML and CSV export utilities.
- Write sample department data and demo scenario JSON files.
- Build and run unit test suite (`tests/test_main.cpp`).

### Phase 5: CLI Application
- Build console menu app wrapping generator, disruption repair, validator, scorer, and export.
- Verify CLI runs cleanly.

### Phase 6: Qt 6 Widgets Desktop GUI
- Implement modern responsive UI (minimum 1100x700):
  - Sidebar navigation with collapsible mode.
  - Top bar with college title, file indicator, theme switcher.
  - 8 pages: Dashboard, Data Management, Timetable Grid, Disruptions, Repair Report, Validation, Score Breakdown, Settings.
  - Custom QPainter widgets for Score Gauge and Breakdown bars.
  - Color-coded chips, tooltips, right-click context menu (Pin/Unpin, Details).
  - Background solver worker thread (`QThread`) with progress feedback.
  - HTML/CSV/PDF export functionality.

### Phase 7: Verification & Screenshots
- Launch GUI in Light and Dark themes.
- Test workflows: Load sample -> Generate -> Add disruption -> Repair -> Validate -> Export.
- Capture screenshots of every page and verify visual polish.

### Phase 8: Final Documentation & Viva Preparation
- Write comprehensive `README.md` including viva table of C++ concepts (polymorphism, templates, smart pointers, RAII, operator overloading, lambdas, custom exceptions).
