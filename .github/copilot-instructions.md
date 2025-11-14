# Turtlebot3-UE Project AI Agent Instructions

## Project Overview
This is an Unreal Engine 5.7 C++ project for simulating ROS2-enabled Turtlebot3 robots. It integrates ROS2 communication into UE through two git submodule plugins: **rclUE** (ROS2-UE bridge) and **RapyutaSimulationPlugins** (robot simulation utilities).

## Architecture

### Three-Repository Structure
- **Main**: `turtlebot3-UE` - Example maps and robot implementations
- **Plugin 1**: `Plugins/rclUE` - Core ROS2 integration (submodule)
- **Plugin 2**: `Plugins/RapyutaSimulationPlugins` - Robot components and sensors (submodule)

### Key Components
- **UROS2NodeComponent**: Central ROS2 node attached to actors
- **UROS2Subsystem**: UGameInstanceSubsystem managing ROS2 lifecycle via `UROS2Support`
- **Publishers/Subscribers**: Created via `ROS2_CREATE_PUBLISHER`/`ROS2_CREATE_SUBSCRIBER` macros
- **Message Classes**: UObject wrappers around ROS2 messages (e.g., `UROS2LaserScanMsg`, `UROS2TwistMsg`)

### Data Flow
1. UE sensors (e.g., `URR2DLidarComponent`) → `SetROS2Msg()` → `UROS2GenericMsg`
2. ROS2 publishers tick in game loop via `SensorPublisher->Publish()`
3. External ROS2 nodes communicate via FastDDS middleware
4. Subscribers receive callbacks updating UE actor states

## Critical UE 5.7 Patterns

### API Breaking Changes
- `GetLineBatcher(UWorld::ELineBatcherType::WorldPersistent)` replaces `PersistentLineBatcher`
- `std::is_same_v<>` from `<type_traits>` replaces `TIsSame<>` (Core module only)
- `FindObject<T>(nullptr, path, EFindObjectFlags::ExactClass)` replaces bool parameter
- `MovementComp->` → `NavMovementInterface->` for AI movement
- Format string validation is stricter - all `FString::Printf` args must match

## Build System

### Environment Setup
```bash
export UE5_DIR=/path/to/Unreal_Engine_5.7.0
```

### Build Commands
```bash
./update_project_files.sh           # Generate .sln/.code-workspace
./make_editor.sh                     # Compile (wrapper around UnrealBuildTool)
./run_editor.sh false                # Launch editor without DDS server
./run_editor.sh true                 # Launch with FastDDS discovery server
```

### Build System Details
- Uses **Clang 20.1.8** from UE's bundled toolchain (`v26_clang-20.1.8-rockylinux8`)
- C++20 standard (`CppStandard = CppStandardVersion.Cpp20` in `.Build.cs`)
- BuildSettingsVersion **V6** for UE 5.7 (updated from V2/V5)
- Unreal Build Accelerator (UBA) enabled for parallel compilation

## ROS2 Integration

### FastDDS Discovery Server
```bash
source fastdds_setup.sh  # Sets ROS_DISCOVERY_SERVER env vars
./run_discovery_service.sh
```
Skip for simple tests: `./run_editor.sh false`

### System Dependencies
```bash
sudo apt-get install -y libspdlog-dev libfmt-dev  # Required for rclUE
```
Missing these causes: `"The game module 'turtlebot3' could not be loaded"` with dlopen errors.

### ROS2 Node Creation Pattern
```cpp
// In robot controller BeginPlay():
if (ROS2Interface && ROS2Interface->RobotROS2Node && 
    ROS2Interface->RobotROS2Node->State == UROS2State::Initialized)
{
    ROS2_CREATE_SUBSCRIBER(ROS2Interface->RobotROS2Node,
                          this, TEXT("GoalTopic"),
                          UROS2PoseStampedMsg::StaticClass(),
                          &ThisClass::GoalCallback);
}
```

## Version Matrix

| UE Version | Ubuntu | ROS2 Distro | rclUE Branch        | RapyutaSim Branch | turtlebot3-UE Branch |
|------------|--------|-------------|---------------------|-------------------|----------------------|
| 5.7.0      | 24.04  | Jazzy       | UE5.7_devel_jazzy   | UE5.7             | jazzy_UE5.7          |
| 5.5.3      | 22.04  | Humble      | UE5.5_devel_humble  | UE5.5             | jammy_UE5.5          |

## Unit Conventions
- **UE uses centimeters**, ROS2 uses meters
- Convert via `URRConversionUtils::MeterToCentimeter()` / `CentimeterToMeter()`
- See: `Plugins/RapyutaSimulationPlugins/Source/.../RRConversionUtils.h`

## Testing
```bash
# Run navigation tests with Nav2
./ExternalTest/run_local_sim_tb3_tests.sh \
    <ue_exe> Turtlebot3AutoTest burger burger0 "0.0,0.0,0.1" "0.0,0.0,0.0"
```

## Git Submodule Workflow
```bash
git submodule update --init --recursive  # Initial checkout
git submodule foreach git pull origin <branch>  # Update all
cd Plugins/rclUE && git checkout UE5.7_devel_jazzy  # Switch branches
```

## Common Patterns

### Sensor Publishing
```cpp
class URRLidarComponent : public URRBaseSensorPublisherComponent {
    virtual void SetROS2Msg(UROS2GenericMsg* InMessage) override {
        CastChecked<UROS2LaserScanMsg>(InMessage)->SetMsg(GetROS2Data());
    }
};
```

### UPROPERTY Limitations
- `rcl_*` types and `void*` cannot use `UPROPERTY()` (not GC-safe)
- Wrap in C++ classes, expose via Blueprint-callable functions
- Example: `UROS2Support` wraps `rclc_support_t`

## Pre-Commit Hooks
```bash
./setup_pre_commit.sh  # Installs for all submodules
```

## Documentation
- rclUE: https://rclUE.readthedocs.io/en/devel/
- RapyutaSimPlugins: https://RapyutaSimulationPlugins.readthedocs.io/en/devel/
