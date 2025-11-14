# Unreal Engine 5.7.0 Migration Plan
## turtlebot3-UE Project Migration

**Date Created:** November 14, 2025  
**Target:** Migrate from UE 5.5 to UE 5.7.0  
**Branch:** jazzy_UE5.7  
**Status:** Planning Phase

---

## Project Scope

This migration involves updating three repositories:
1. **Main Repository**: `turtlebot3-UE` (this repository)
2. **Submodule 1**: `rclUE` → `https://github.com/Zartris/rclUE/tree/UE5.7_devel_jazzy`
3. **Submodule 2**: `RapyutaSimulationPlugins` → `https://github.com/Zartris/RapyutaSimulationPlugins/tree/UE5.7`

**Important Note:** Both submodule repositories are currently on UE 5.5 and require migration to UE 5.7.0.

---

## Current State Analysis

### Main Repository
- **Current Engine:** UE 5.5 (from `turtlebot3.uproject`)
- **Current Branch:** `jazzy_UE5.7`
- **Working Directory:** Clean ✅

### Submodules Status
```
04627ffcf36f7a527881b1583fa3710608398689 Plugins/RapyutaSimulationPlugins (remotes/origin/UE4-167-g04627ffc)
ce236ae296f903c513ad50ce5c9bb232365dd0ca Plugins/rclUE (remotes/origin/UE4-47-gce236ae2)
```

### Current Submodule Configuration
```ini
[submodule "Plugins/rclUE"]
        path = Plugins/rclUE
        url = https://github.com/rapyuta-robotics/rclUE.git
[submodule "Plugins/RapyutaSimulationPlugins"]
        path = Plugins/RapyutaSimulationPlugins
```

---

## Migration Strategy

### Phase 1: Repository Setup & Preparation

#### 1.1 Update Main Project Engine Association
- [ ] **Task:** Update Engine Association to UE 5.7
- **File:** `turtlebot3.uproject`
- **Change:** `"EngineAssociation": "5.5"` → `"EngineAssociation": "5.7"`

#### 1.2 Update Submodule Configuration
- [ ] **Task:** Update rclUE submodule URL and branch
- **File:** `.gitmodules`
- **Change:** Point to `https://github.com/Zartris/rclUE.git`

- [ ] **Task:** Update RapyutaSimulationPlugins submodule URL and branch
- **File:** `.gitmodules`
- **Change:** Point to `https://github.com/Zartris/RapyutaSimulationPlugins.git`

#### 1.3 Sync Repository Configuration
- [ ] **Task:** Sync submodules to new repositories
- **Commands:**
  ```bash
  git submodule sync
  git submodule update --init --recursive
  ```

- [ ] **Task:** Checkout correct branches in submodules
- **Commands:**
  ```bash
  cd Plugins/rclUE && git checkout UE5.7_devel_jazzy
  cd ../RapyutaSimulationPlugins && git checkout UE5.7
  ```

### Phase 2: Plugin Migration (Critical Phase)

#### 2.1 rclUE Plugin Migration
- [ ] **Task:** Update rclUE plugin to UE 5.7
- **Location:** `Plugins/rclUE/`
- **Focus Areas:**
  - Update `.uplugin` file engine version requirements
  - Review and update C++ API usage for UE 5.7 compatibility
  - Check ROS2 integration compatibility with UE 5.7
  - Update build scripts and module dependencies
  - Test compilation of plugin standalone

**Key Files to Review:**
- `rclUE.uplugin`
- `Source/rclUE/rclUE.Build.cs`
- All C++ header and source files using UE APIs

#### 2.2 RapyutaSimulationPlugins Migration
- [ ] **Task:** Update RapyutaSimulationPlugins to UE 5.7
- **Location:** `Plugins/RapyutaSimulationPlugins/`
- **Focus Areas:**
  - Update `.uplugin` file engine version requirements
  - Review simulation-specific API changes
  - Update physics and rendering system integration
  - Check for deprecated simulation functions
  - Test compilation of plugin standalone

**Key Files to Review:**
- `RapyutaSimulationPlugins.uplugin`
- Source files with simulation and physics code
- Blueprint function libraries

### Phase 3: Integration & Testing

#### 3.1 Project File Regeneration
- [ ] **Task:** Update project files for UE 5.7
- **Command:** `./update_project_files.sh`
- **Purpose:** Generate UE 5.7 compatible project files

#### 3.2 Compilation Testing
- [ ] **Task:** Test plugin compilation individually
- **Process:**
  1. Compile rclUE plugin in isolation
  2. Compile RapyutaSimulationPlugins in isolation
  3. Identify and fix plugin-specific issues

- [ ] **Task:** Test main project compilation with UE 5.7
- **Process:**
  1. Full project compilation
  2. Identify integration issues
  3. Resolve any remaining compilation errors

#### 3.3 Configuration Review
- [ ] **Task:** Update configuration files if needed
- **Files to Review:**
  - `Config/DefaultEngine.ini`
  - `Config/DefaultGame.ini`
  - `Config/DefaultEditor.ini`
- **Check for:** UE 5.7 specific settings, deprecated options

### Phase 4: Version Control & Finalization

#### 4.1 Submodule Repository Commits
- [ ] **Task:** Commit plugin updates to submodule repos
- **Process:**
  1. Commit rclUE changes to `UE5.7_devel_jazzy` branch
  2. Commit RapyutaSimulationPlugins changes to `UE5.7` branch
  3. Push both repositories

#### 4.2 Main Repository Commit
- [ ] **Task:** Commit main project updates
- **Process:**
  1. Stage submodule reference updates
  2. Stage any main project file changes
  3. Commit to `jazzy_UE5.7` branch

#### 4.3 Testing & Validation
- [ ] **Task:** Test full functionality
- **Scope:**
  - Basic project loading in UE 5.7
  - ROS2 communication functionality
  - Simulation features
  - Blueprint compilation
  - Runtime stability

---

## Known Potential Issues

### UE 5.5 → 5.7 Breaking Changes to Watch For:
1. **API Deprecations:** Functions marked deprecated in 5.5 may be removed in 5.7
2. **Build System Changes:** Module dependencies and build configurations
3. **Blueprint API Changes:** Node signature changes
4. **Physics System Updates:** Chaos physics engine updates
5. **Rendering Pipeline Changes:** Material and lighting system updates

### ROS2 Specific Considerations:
1. **Threading Model:** UE 5.7 threading changes affecting ROS2 callbacks
2. **Memory Management:** Smart pointer usage changes
3. **Plugin Loading Order:** Dependency resolution changes

---

## Prerequisites Checklist

- [ ] Unreal Engine 5.7.0 installed and accessible
- [ ] Git access to both submodule repositories
- [ ] Development environment set up for UE 5.7 compilation
- [ ] Backup of current working state (if needed)

---

## Emergency Rollback Plan

If critical issues are encountered:
1. Revert `.gitmodules` to original configuration
2. Reset submodules: `git submodule update --init --recursive`
3. Revert `turtlebot3.uproject` engine association to 5.5
4. Regenerate project files for UE 5.5

---

## Progress Tracking

Use this checklist to track progress through each phase:

**Phase 1: Setup** ⏳
- [ ] Engine association updated
- [ ] Submodule URLs updated
- [ ] Repositories synced
- [ ] Branches checked out

**Phase 2: Plugin Migration** ⏳
- [ ] rclUE updated for UE 5.7
- [ ] RapyutaSimulationPlugins updated for UE 5.7
- [ ] Both plugins compile successfully

**Phase 3: Integration** ⏳
- [ ] Project files regenerated
- [ ] Individual plugin compilation successful
- [ ] Full project compilation successful
- [ ] Configuration files reviewed

**Phase 4: Finalization** ⏳
- [ ] Submodule changes committed and pushed
- [ ] Main project changes committed
- [ ] Full functionality testing completed

---

## Notes & Observations

*Use this section to document any issues, workarounds, or important findings during the migration process.*

---

**Migration Plan Version:** 1.0  
**Last Updated:** November 14, 2025