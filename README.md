sequenceDiagram
    participant ROSNode
    participant TebPlannerROS
    participant costmap_ros
    participant odom_helper
    participant planner
    participant visualization
    participant costmap_model

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

    activate TebPlannerROS
    TebPlannerROS -->> TebPlannerROS: Check for initialization success

    activate TebPlannerROS
    TebPlannerROS -->> TebPlannerROS: Transform global plan to local frame
    TebPlannerROS -->> costmap_ros: getRobotPose
    activate costmap_ros
    costmap_ros -->> TebPlannerROS: robot_pose

    TebPlannerROS -->> planner: pruneGlobalPlan
    activate planner
    planner -->> planner: Prune global plan spatially
    planner -->> planner: Transform global plan to local frame
    deactivate planner

    TebPlannerROS -->> planner: transformGlobalPlan
    activate planner
    planner -->> planner: Transform global plan to local frame
    deactivate planner

    TebPlannerROS -->> TebPlannerROS: Update via-points container

    deactivate costmap_ros
    deactivate TebPlannerROS

    activate TebPlannerROS
    TebPlannerROS -->> costmap_model: updateObstacleContainerWithCostmap
    activate costmap_model
    costmap_model -->> TebPlannerROS: Updated obstacle container

    TebPlannerROS -->> TebPlannerROS: Update obstacle container with custom obstacles

    deactivate TebPlannerROS
    deactivate costmap_model

    activate TebPlannerROS
    TebPlannerROS ->> planner: plan(transformed_plan, &robot_vel_, cfg_.goal_tolerance.free_goal_vel)
    activate planner

    planner -->> TebPlannerROS: success (bool)

    deactivate planner
    deactivate TebPlannerROS

    activate TebPlannerROS
    TebPlannerROS -->> TebPlannerROS: Check for success

    activate TebPlannerROS
    TebPlannerROS -->> planner: getVelocityCommand
    activate planner
    planner -->> TebPlannerROS: velocity_command

    deactivate planner
    deactivate TebPlannerROS

    activate TebPlannerROS
    TebPlannerROS -->> TebPlannerROS: Saturate velocity

    activate TebPlannerROS
    TebPlannerROS -->> TebPlannerROS: Convert rot-vel to steering angle

    deactivate TebPlannerROS

    activate TebPlannerROS
    TebPlannerROS -->> TebPlannerROS: Store last command

    activate TebPlannerROS
    TebPlannerROS -->> planner: visualize
    activate planner
    planner -->> visualization: Visualize obstacles, via points, global plan
    deactivate planner
    deactivate TebPlannerROS
