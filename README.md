```mermaid
sequenceDiagram
    participant ROSNode
    participant TebPlannerROS
    participant OtherComponents
    participant costmap_ros
    participant odom_helper
    participant planner
    participant visualization

    Note over ROSNode: Initialization

    activate ROSNode
    ROSNode ->> TebPlannerROS: computeVelocityCommands
    activate TebPlannerROS

    TebPlannerROS ->> ROSNode: ROS_ERROR("teb_path_smooth_controller has not been initialized...")
    Note over ROSNode: Set message and return status

    deactivate TebPlannerROS
    deactivate ROSNode

    activate TebPlannerROS
    TebPlannerROS ->> costmap_ros: getRobotPose
    activate costmap_ros
    costmap_ros -->> TebPlannerROS: robot_pose

    TebPlannerROS ->> odom_helper: getRobotVel
    activate odom_helper
    odom_helper -->> TebPlannerROS: robot_vel_tf

    TebPlannerROS ->> planner: plan(transformed_plan, &robot_vel_, cfg_.goal_tolerance.free_goal_vel)
    activate planner

    planner -->> TebPlannerROS: success (bool)

    deactivate planner
    deactivate odom_helper
    deactivate costmap_ros
    deactivate TebPlannerROS

    Note over TebPlannerROS: Check feasibility
    activate TebPlannerROS
    TebPlannerROS ->> planner: isTrajectoryFeasible
    activate planner
    planner -->> TebPlannerROS: feasible (bool)

    deactivate planner
    deactivate TebPlannerROS

    Note over TebPlannerROS: Get velocity command
    activate TebPlannerROS
    TebPlannerROS ->> planner: getVelocityCommand
    activate planner
    planner -->> TebPlannerROS: velocity_command

    deactivate planner
    deactivate TebPlannerROS

    Note over TebPlannerROS: Saturate velocity
    activate TebPlannerROS
    TebPlannerROS -->> TebPlannerROS: saturateVelocity

    Note over TebPlannerROS: Convert rot-vel to steering angle
    TebPlannerROS -->> TebPlannerROS: convertTransRotVelToSteeringAngle

    deactivate TebPlannerROS

    activate TebPlannerROS
    TebPlannerROS -->> planner: visualize
    activate planner
    planner -->> visualization: visualize obstacles, via points, global plan
    deactivate planner
    deactivate TebPlannerROS

