---
sidebar_position: 2
title: "LLM-Based Cognitive Planning"
---

# LLM-Based Cognitive Planning for Humanoid Robotics

## Learning Objectives

By the end of this chapter, you will be able to:
- Understand the role of Large Language Models (LLMs) in cognitive planning for humanoid robots
- Implement LLM-based task decomposition and planning systems
- Integrate LLMs with ROS 2 for robot behavior planning
- Design effective prompts for robot planning tasks
- Evaluate and validate LLM-generated plans for safety and feasibility
- Deploy LLM-based cognitive planning systems on humanoid robots

## Introduction to LLM-Based Cognitive Planning

Large Language Models (LLMs) have emerged as powerful tools for cognitive planning in robotics, enabling robots to understand complex natural language instructions and generate sophisticated action plans. For humanoid robots, LLM-based cognitive planning bridges the gap between high-level human instructions and low-level robot control, allowing for more intuitive and flexible robot behavior.

### What is Cognitive Planning?

Cognitive planning in robotics refers to the process of generating sequences of actions that enable a robot to achieve high-level goals. This involves:

1. **Goal Interpretation**: Understanding the desired outcome from human instructions
2. **Task Decomposition**: Breaking complex tasks into manageable subtasks
3. **Action Sequencing**: Ordering actions to achieve the goal efficiently
4. **Constraint Handling**: Ensuring plans respect physical and safety constraints
5. **Adaptation**: Adjusting plans based on environmental changes or failures

### Benefits of LLM-Based Planning

- **Natural Language Understanding**: Direct interpretation of human instructions
- **Common-Sense Reasoning**: LLMs can apply general knowledge to novel situations
- **Flexibility**: Ability to handle diverse and complex instructions
- **Scalability**: Can leverage pre-trained knowledge from large datasets
- **Learning**: Can improve with experience and feedback

## LLM Architectures for Robotics

### Overview of LLM Architectures

Several LLM architectures are suitable for robotic planning:

1. **Transformer-based Models**: GPT, Claude, Llama series
2. **Instruction-Tuned Models**: Models fine-tuned for following instructions
3. **Multimodal Models**: Models that can process both text and images
4. **Specialized Robot Models**: Models trained specifically for robotic tasks

### Choosing the Right LLM for Robotics

When selecting an LLM for cognitive planning, consider:

- **Latency Requirements**: Real-time vs. batch processing
- **Safety Criticality**: Open vs. closed models
- **Computational Resources**: Local vs. cloud processing
- **Privacy Requirements**: Data handling and storage
- **Domain Specificity**: General vs. robotics-focused models

### LLM Integration Architecture

```python
import openai
import asyncio
import json
from typing import Dict, List, Optional
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from geometry_msgs.msg import Pose
import time

class LLMCognitivePlanner(Node):
    def __init__(self):
        super().__init__('llm_cognitive_planner')

        # LLM configuration
        self.api_key = "YOUR_API_KEY"  # In production, use environment variables
        self.model = "gpt-4-turbo"  # or another suitable model
        self.temperature = 0.3  # Lower for more deterministic output

        # Publishers and subscribers
        self.plan_request_sub = self.create_subscription(
            String, 'plan_requests', self.plan_request_callback, 10
        )
        self.plan_output_pub = self.create_publisher(String, 'generated_plans', 10)

        # Robot state information
        self.robot_state = {
            'location': 'unknown',
            'battery_level': 100.0,
            'available_arms': ['left', 'right'],
            'current_task': None,
            'known_objects': [],
            'known_locations': []
        }

        # Planning history
        self.planning_history = []

    def plan_request_callback(self, msg):
        """Process planning request from high-level task"""
        request = json.loads(msg.data)
        task_description = request.get('task', '')
        priority = request.get('priority', 'normal')

        self.get_logger().info(f'Received planning request: {task_description}')

        # Generate plan using LLM
        plan = self.generate_plan(task_description, priority)

        if plan:
            # Publish the generated plan
            plan_msg = String()
            plan_msg.data = json.dumps({
                'task': task_description,
                'plan': plan,
                'timestamp': time.time()
            })
            self.plan_output_pub.publish(plan_msg)

            # Update planning history
            self.planning_history.append({
                'task': task_description,
                'plan': plan,
                'timestamp': time.time()
            })

    def generate_plan(self, task_description: str, priority: str = 'normal') -> Optional[List[Dict]]:
        """Generate a plan using LLM"""
        try:
            # Construct the prompt for the LLM
            prompt = self.construct_planning_prompt(task_description, priority)

            # Call the LLM API
            response = openai.ChatCompletion.create(
                model=self.model,
                messages=[
                    {"role": "system", "content": self.get_system_prompt()},
                    {"role": "user", "content": prompt}
                ],
                temperature=self.temperature,
                max_tokens=1000
            )

            # Extract the plan from the response
            plan_text = response.choices[0].message['content'].strip()

            # Parse the plan
            plan = self.parse_plan(plan_text)

            # Validate the plan
            if self.validate_plan(plan):
                return plan
            else:
                self.get_logger().warn('Generated plan failed validation')
                return None

        except Exception as e:
            self.get_logger().error(f'Error generating plan: {str(e)}')
            return None

    def construct_planning_prompt(self, task_description: str, priority: str) -> str:
        """Construct the prompt for the LLM"""
        return f"""
        You are a cognitive planning system for a humanoid robot. Your task is to generate a detailed plan to accomplish the following goal:

        TASK: {task_description}

        ROBOT CAPABILITIES:
        - Can navigate to different locations
        - Has two arms for manipulation
        - Can pick up and place objects
        - Can open/close doors
        - Can communicate with humans

        CURRENT ROBOT STATE:
        - Location: {self.robot_state['location']}
        - Battery level: {self.robot_state['battery_level']}%
        - Available arms: {', '.join(self.robot_state['available_arms'])}
        - Known objects: {', '.join(self.robot_state['known_objects']) if self.robot_state['known_objects'] else 'None'}
        - Known locations: {', '.join(self.robot_state['known_locations']) if self.robot_state['known_locations'] else 'None'}

        PLANNING REQUIREMENTS:
        1. Break down the task into specific, executable actions
        2. Consider the robot's current state and capabilities
        3. Account for safety and feasibility
        4. Output the plan in JSON format with the following structure:
           {{
             "plan": [
               {{
                 "step": 1,
                 "action": "action_type",
                 "description": "detailed description of the action",
                 "parameters": {{"param1": "value1", "param2": "value2"}},
                 "expected_outcome": "what should happen after this action"
               }}
             ]
           }}

        PRIORITY LEVEL: {priority}

        Generate the plan in the required JSON format:
        """

    def get_system_prompt(self) -> str:
        """Get the system prompt for the LLM"""
        return """
        You are an expert cognitive planning system for humanoid robots. Your role is to generate detailed, executable plans that:
        1. Are safe and feasible given robot capabilities
        2. Account for environmental constraints
        3. Consider the robot's current state
        4. Include error handling and recovery strategies
        5. Are broken down into specific, executable steps

        Always respond with a valid JSON object containing the plan. Do not include any text outside the JSON.
        """

    def parse_plan(self, plan_text: str) -> Optional[List[Dict]]:
        """Parse the plan from LLM response"""
        try:
            # Clean the response to extract JSON
            plan_text = plan_text.strip()

            # Look for JSON between ```json and ``` or just extract JSON
            if '```json' in plan_text:
                start = plan_text.find('```json') + 7
                end = plan_text.find('```', start)
                plan_text = plan_text[start:end].strip()
            elif '```' in plan_text:
                start = plan_text.find('```') + 3
                end = plan_text.find('```', start)
                plan_text = plan_text[start:end].strip()

            # Parse JSON
            plan_data = json.loads(plan_text)

            # Extract the plan if it's nested
            if 'plan' in plan_data:
                return plan_data['plan']
            else:
                # Assume the entire response is the plan
                return plan_data if isinstance(plan_data, list) else None

        except json.JSONDecodeError as e:
            self.get_logger().error(f'Error parsing plan JSON: {str(e)}')
            return None
        except Exception as e:
            self.get_logger().error(f'Error parsing plan: {str(e)}')
            return None

    def validate_plan(self, plan: List[Dict]) -> bool:
        """Validate the generated plan"""
        if not plan:
            return False

        for step in plan:
            if not isinstance(step, dict):
                return False

            required_keys = ['step', 'action', 'description']
            for key in required_keys:
                if key not in step:
                    return False

            # Validate action types
            valid_actions = [
                'navigate', 'pick_up', 'place', 'open_door', 'close_door',
                'communicate', 'wait', 'detect_object', 'move_arm', 'stop'
            ]

            if step['action'] not in valid_actions:
                self.get_logger().warn(f'Invalid action type: {step["action"]}')
                return False

        return True

def main(args=None):
    rclpy.init(args=args)
    planner = LLMCognitivePlanner()

    try:
        rclpy.spin(planner)
    except KeyboardInterrupt:
        pass
    finally:
        planner.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Prompt Engineering for Robot Planning

### Effective Prompt Design

Prompt engineering is crucial for getting reliable outputs from LLMs for robot planning:

```python
class PromptEngineer:
    def __init__(self):
        self.base_context = {
            'robot_type': 'humanoid',
            'environment': 'indoor',
            'safety_constraints': True,
            'real_world': True
        }

    def create_task_decomposition_prompt(self, high_level_task: str, context: Dict = None) -> str:
        """Create a prompt for decomposing high-level tasks"""
        ctx = context or self.base_context

        return f"""
        Decompose the following high-level task into specific, executable subtasks for a {ctx['robot_type']} robot operating in a {ctx['environment']} environment.

        HIGH-LEVEL TASK: {high_level_task}

        ROBOT CAPABILITIES:
        - Navigation (move to specific locations)
        - Manipulation (pick up, place, and manipulate objects)
        - Perception (detect and recognize objects)
        - Communication (speak and listen)

        ENVIRONMENT CONSTRAINTS:
        - Indoor environment with furniture, doors, and obstacles
        - Standard human-scale spaces
        - Potentially dynamic (humans moving around)

        SAFETY REQUIREMENTS:
        - Avoid collisions with humans and obstacles
        - Respect personal space
        - Handle objects safely
        - Operate within physical limits

        OUTPUT FORMAT:
        Provide the decomposition as a numbered list with each subtask containing:
        1. A clear action description
        2. Required preconditions
        3. Expected outcomes
        4. Potential challenges or failure modes

        SUBTASK DECOMPOSITION:
        """

    def create_safety_validation_prompt(self, plan: List[Dict]) -> str:
        """Create a prompt for validating plan safety"""
        plan_str = json.dumps(plan, indent=2)

        return f"""
        Analyze the following robot action plan for safety issues and potential hazards:

        PLAN:
        {plan_str}

        SAFETY CRITERIA:
        1. Does the plan involve unsafe interactions with humans?
        2. Could any actions cause damage to the robot or environment?
        3. Are there potential collision risks?
        4. Does the plan respect privacy and social norms?
        5. Are there any physically impossible actions?

        OUTPUT:
        1. List any safety issues identified
        2. Rate the overall safety (Safe/Conditionally Safe/Unsafe)
        3. Suggest modifications to address safety concerns
        4. Identify critical safety checks needed during execution

        ANALYSIS:
        """

    def create_feasibility_check_prompt(self, action: Dict, robot_state: Dict) -> str:
        """Create a prompt for checking action feasibility"""
        return f"""
        Determine if the following action is feasible given the robot's current state:

        ACTION: {action}
        ROBOT STATE: {robot_state}

        FEASIBILITY FACTORS:
        1. Physical capabilities
        2. Environmental constraints
        3. Safety considerations
        4. Resource availability (battery, time)

        OUTPUT:
        - Feasible: Yes/No/Conditionally
        - Reasoning: Brief explanation
        - Required conditions: What needs to be true for this to work
        - Alternative suggestions: If not feasible, what alternatives exist

        FEASIBILITY ASSESSMENT:
        """
```

## Integration with ROS 2

### ROS 2 Action Server for LLM Planning

```python
import rclpy
from rclpy.action import ActionServer, GoalResponse, CancelResponse
from rclpy.node import Node
from rclpy.executors import MultiThreadedExecutor
from rclpy.callback_groups import ReentrantCallbackGroup
from std_msgs.msg import String
from geometry_msgs.msg import Pose
import json
import threading
import time

class LLMPlanningActionServer(Node):
    def __init__(self):
        super().__init__('llm_planning_action_server')

        # Create action server
        self._action_server = ActionServer(
            self,
            Planning,
            'llm_plan_action',
            execute_callback=self.execute_plan_callback,
            goal_callback=self.goal_callback,
            cancel_callback=self.cancel_callback,
            callback_group=ReentrantCallbackGroup()
        )

        # Publishers for intermediate results
        self.plan_feedback_pub = self.create_publisher(String, 'plan_feedback', 10)

        # LLM planner instance
        self.llm_planner = LLMCognitivePlanner()

    def goal_callback(self, goal_request):
        """Accept or reject goal request"""
        self.get_logger().info('Received planning goal request')
        return GoalResponse.ACCEPT

    def cancel_callback(self, goal_handle):
        """Accept or reject cancel request"""
        self.get_logger().info('Received cancel planning request')
        return CancelResponse.ACCEPT

    async def execute_plan_callback(self, goal_handle):
        """Execute the planning goal"""
        self.get_logger().info('Executing planning goal')

        feedback_msg = Planning.Feedback()
        result = Planning.Result()

        try:
            # Extract task from goal
            task_description = goal_handle.request.task_description
            priority = goal_handle.request.priority

            # Generate plan using LLM
            plan = self.llm_planner.generate_plan(task_description, priority)

            if plan:
                # Publish feedback
                feedback_msg.status = "Plan generated successfully"
                goal_handle.publish_feedback(feedback_msg)

                # Convert plan to result
                result.plan = json.dumps(plan)
                result.success = True
                result.message = "Plan generated successfully"

                goal_handle.succeed()
            else:
                result.success = False
                result.message = "Failed to generate valid plan"
                goal_handle.abort()

        except Exception as e:
            self.get_logger().error(f'Error in planning execution: {str(e)}')
            result.success = False
            result.message = f"Planning failed: {str(e)}"
            goal_handle.abort()

        return result

def main(args=None):
    rclpy.init(args=args)

    # Create node with action server
    action_server = LLMPlanningActionServer()

    # Create multi-threaded executor
    executor = MultiThreadedExecutor(num_threads=4)
    executor.add_node(action_server)

    try:
        executor.spin()
    except KeyboardInterrupt:
        pass
    finally:
        action_server.destroy_node()
        rclpy.shutdown()
```

### Plan Execution and Monitoring

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from action_msgs.msg import GoalStatus
from rclpy.action import ActionClient
from geometry_msgs.msg import Pose
import json
import time
from enum import Enum

class ExecutionState(Enum):
    IDLE = "idle"
    EXECUTING = "executing"
    PAUSED = "paused"
    FAILED = "failed"
    COMPLETED = "completed"

class PlanExecutor(Node):
    def __init__(self):
        super().__init__('plan_executor')

        # Publishers and subscribers
        self.plan_sub = self.create_subscription(
            String, 'generated_plans', self.plan_callback, 10
        )
        self.status_pub = self.create_publisher(String, 'execution_status', 10)
        self.feedback_pub = self.create_publisher(String, 'execution_feedback', 10)

        # Robot control publishers
        self.nav_goal_pub = self.create_publisher(Pose, 'navigation_goal', 10)
        self.manipulation_cmd_pub = self.create_publisher(String, 'manipulation_commands', 10)

        # Navigation action client
        self.nav_client = ActionClient(self, NavigateToPose, 'navigate_to_pose')

        # Execution state
        self.current_plan = None
        self.current_step_index = 0
        self.execution_state = ExecutionState.IDLE
        self.last_execution_time = None

    def plan_callback(self, msg):
        """Receive and start executing a plan"""
        plan_data = json.loads(msg.data)
        plan = plan_data['plan']

        if self.execution_state == ExecutionState.IDLE:
            self.current_plan = plan
            self.current_step_index = 0
            self.execution_state = ExecutionState.EXECUTING
            self.last_execution_time = time.time()

            self.get_logger().info(f'Starting execution of plan with {len(plan)} steps')
            self.execute_next_step()
        else:
            self.get_logger().warn('Plan executor busy, cannot start new plan')

    def execute_next_step(self):
        """Execute the next step in the current plan"""
        if not self.current_plan or self.current_step_index >= len(self.current_plan):
            # Plan completed
            self.execution_state = ExecutionState.COMPLETED
            self.publish_status("Plan completed successfully")
            return

        current_step = self.current_plan[self.current_step_index]
        self.get_logger().info(f'Executing step {self.current_step_index + 1}: {current_step["action"]}')

        # Execute based on action type
        success = self.execute_action(current_step)

        if success:
            # Move to next step
            self.current_step_index += 1

            # Check if plan is complete
            if self.current_step_index >= len(self.current_plan):
                self.execution_state = ExecutionState.COMPLETED
                self.publish_status("Plan completed successfully")
            else:
                # Schedule next step execution
                timer = self.create_timer(0.1, self.execute_next_step)
        else:
            # Execution failed
            self.execution_state = ExecutionState.FAILED
            self.publish_status(f"Plan execution failed at step {self.current_step_index + 1}")

    def execute_action(self, step: Dict) -> bool:
        """Execute a single action step"""
        action_type = step['action']

        try:
            if action_type == 'navigate':
                return self.execute_navigation(step)
            elif action_type == 'pick_up':
                return self.execute_pickup(step)
            elif action_type == 'place':
                return self.execute_place(step)
            elif action_type == 'communicate':
                return self.execute_communication(step)
            elif action_type == 'wait':
                return self.execute_wait(step)
            elif action_type == 'detect_object':
                return self.execute_object_detection(step)
            elif action_type == 'move_arm':
                return self.execute_arm_movement(step)
            elif action_type == 'stop':
                return self.execute_stop(step)
            else:
                self.get_logger().error(f'Unknown action type: {action_type}')
                return False
        except Exception as e:
            self.get_logger().error(f'Error executing action {action_type}: {str(e)}')
            return False

    def execute_navigation(self, step: Dict) -> bool:
        """Execute navigation action"""
        params = step.get('parameters', {})
        target_location = params.get('location')

        if not target_location:
            self.get_logger().error('Navigation action missing location parameter')
            return False

        # In a real system, this would look up coordinates for the location
        # For this example, we'll use placeholder coordinates
        goal_msg = NavigateToPose.Goal()
        goal_msg.pose.header.frame_id = 'map'

        # Placeholder coordinates - in real system, these would come from a map
        if target_location == 'kitchen':
            goal_msg.pose.pose.position.x = 2.0
            goal_msg.pose.pose.position.y = 1.0
        elif target_location == 'living_room':
            goal_msg.pose.pose.position.x = 0.0
            goal_msg.pose.pose.position.y = 0.0
        else:
            # Try to parse coordinates from parameters
            goal_msg.pose.pose.position.x = params.get('x', 0.0)
            goal_msg.pose.pose.position.y = params.get('y', 0.0)

        goal_msg.pose.pose.orientation.w = 1.0

        # Send navigation goal
        self.nav_client.wait_for_server()
        future = self.nav_client.send_goal_async(goal_msg)
        future.add_done_callback(self.navigation_goal_callback)

        return True

    def navigation_goal_callback(self, future):
        """Handle navigation goal response"""
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().info('Navigation goal rejected')
            return

        self.get_logger().info('Navigation goal accepted')
        result_future = goal_handle.get_result_async()
        result_future.add_done_callback(self.navigation_result_callback)

    def navigation_result_callback(self, future):
        """Handle navigation result"""
        result = future.result().result
        status = future.result().status

        if status == GoalStatus.STATUS_SUCCEEDED:
            self.get_logger().info('Navigation succeeded')
            # Continue with next step
            self.execute_next_step()
        else:
            self.get_logger().info(f'Navigation failed with status: {status}')
            self.execution_state = ExecutionState.FAILED

    def execute_pickup(self, step: Dict) -> bool:
        """Execute pickup action"""
        params = step.get('parameters', {})
        object_name = params.get('object')
        location = params.get('location', 'current')

        self.get_logger().info(f'Attempting to pick up {object_name} at {location}')

        # In a real system, this would involve:
        # 1. Navigating to the object location
        # 2. Detecting the object
        # 3. Planning grasp trajectory
        # 4. Executing grasp

        # For this example, we'll just publish a command
        cmd_msg = String()
        cmd_msg.data = json.dumps({
            'action': 'pickup',
            'object': object_name,
            'location': location
        })
        self.manipulation_cmd_pub.publish(cmd_msg)

        return True

    def execute_place(self, step: Dict) -> bool:
        """Execute place action"""
        params = step.get('parameters', {})
        object_name = params.get('object')
        location = params.get('location')

        self.get_logger().info(f'Attempting to place {object_name} at {location}')

        cmd_msg = String()
        cmd_msg.data = json.dumps({
            'action': 'place',
            'object': object_name,
            'location': location
        })
        self.manipulation_cmd_pub.publish(cmd_msg)

        return True

    def execute_communication(self, step: Dict) -> bool:
        """Execute communication action"""
        params = step.get('parameters', {})
        message = params.get('message', '')
        target = params.get('target', 'user')

        self.get_logger().info(f'Communicating: "{message}" to {target}')

        # Publish to text-to-speech system
        tts_msg = String()
        tts_msg.data = message
        # Assuming there's a TTS publisher at this topic
        # tts_pub.publish(tts_msg)

        return True

    def execute_wait(self, step: Dict) -> bool:
        """Execute wait action"""
        params = step.get('parameters', {})
        duration = params.get('duration', 1.0)  # seconds

        self.get_logger().info(f'Waiting for {duration} seconds')

        # In a real implementation, this would use a timer
        # For now, we'll just continue to the next step immediately
        time.sleep(duration)
        return True

    def execute_object_detection(self, step: Dict) -> bool:
        """Execute object detection action"""
        params = step.get('parameters', {})
        object_name = params.get('object', 'any')
        location = params.get('location', 'current')

        self.get_logger().info(f'Detecting {object_name} at {location}')

        # In a real system, this would trigger perception pipeline
        # For this example, we'll assume detection succeeds
        return True

    def execute_arm_movement(self, step: Dict) -> bool:
        """Execute arm movement action"""
        params = step.get('parameters', {})
        arm = params.get('arm', 'both')
        pose = params.get('pose', 'default')

        self.get_logger().info(f'Moving {arm} arm to {pose} pose')

        cmd_msg = String()
        cmd_msg.data = json.dumps({
            'action': 'move_arm',
            'arm': arm,
            'pose': pose
        })
        self.manipulation_cmd_pub.publish(cmd_msg)

        return True

    def execute_stop(self, step: Dict) -> bool:
        """Execute stop action"""
        self.get_logger().info('Stopping current execution')

        # This would stop all robot motion
        self.execution_state = ExecutionState.IDLE
        return True

    def publish_status(self, message: str):
        """Publish execution status"""
        status_msg = String()
        status_msg.data = json.dumps({
            'state': self.execution_state.value,
            'message': message,
            'step': self.current_step_index,
            'total_steps': len(self.current_plan) if self.current_plan else 0
        })
        self.status_pub.publish(status_msg)
        self.get_logger().info(message)
```

## Advanced Planning Techniques

### Hierarchical Task Planning

```python
class HierarchicalPlanner:
    def __init__(self):
        self.task_library = {
            'fetch_object': [
                {'action': 'navigate', 'parameters': {'location': '{target_location}'}},
                {'action': 'detect_object', 'parameters': {'object': '{object_name}'}},
                {'action': 'pick_up', 'parameters': {'object': '{object_name}'}},
                {'action': 'navigate', 'parameters': {'location': '{delivery_location}'}},
                {'action': 'place', 'parameters': {'object': '{object_name}'}}
            ],
            'room_cleanup': [
                {'action': 'navigate', 'parameters': {'location': '{room_name}'}},
                {'action': 'detect_objects', 'parameters': {}},
                {'action': 'for_each_object', 'subtask': 'pick_up_object'},
                {'action': 'navigate', 'parameters': {'location': 'storage_area'}}
            ],
            'greeting': [
                {'action': 'move_arm', 'parameters': {'pose': 'wave'}},
                {'action': 'communicate', 'parameters': {'message': '{greeting_message}'}}
            ]
        }

    def decompose_task(self, task_description: str, context: Dict) -> List[Dict]:
        """Decompose a task using hierarchical planning"""
        # First, try to match to known high-level tasks
        matched_task = self.match_to_known_task(task_description)

        if matched_task:
            # Expand the known task with parameters
            return self.expand_task_template(matched_task, context)

        # If no known task matches, use LLM to create a new plan
        return self.create_new_plan(task_description, context)

    def match_to_known_task(self, task_description: str) -> Optional[str]:
        """Match task description to known task templates"""
        import re

        # Simple keyword matching - in practice, this would use more sophisticated NLP
        task_keywords = {
            'fetch_object': ['fetch', 'get', 'bring', 'pick up', 'retrieve'],
            'room_cleanup': ['clean', 'tidy', 'organize', 'cleanup'],
            'greeting': ['hello', 'hi', 'greet', 'welcome', 'introduce']
        }

        task_lower = task_description.lower()

        for task_name, keywords in task_keywords.items():
            for keyword in keywords:
                if keyword in task_lower:
                    return task_name

        return None

    def expand_task_template(self, task_name: str, context: Dict) -> List[Dict]:
        """Expand a task template with specific parameters"""
        template = self.task_library.get(task_name)
        if not template:
            return []

        expanded_plan = []

        for step_template in template:
            if step_template['action'] == 'for_each_object':
                # Handle special case of iterating over objects
                subtask = step_template['subtask']
                objects = context.get('objects', [])

                for obj in objects:
                    sub_context = {**context, 'object_name': obj}
                    sub_plan = self.expand_task_template(subtask, sub_context)
                    expanded_plan.extend(sub_plan)
            else:
                # Replace placeholders in parameters
                expanded_step = self.replace_placeholders(step_template, context)
                expanded_plan.append(expanded_step)

        return expanded_plan

    def replace_placeholders(self, step: Dict, context: Dict) -> Dict:
        """Replace placeholders in step parameters with context values"""
        import re

        new_step = step.copy()
        if 'parameters' in new_step:
            new_params = {}
            for key, value in new_step['parameters'].items():
                if isinstance(value, str):
                    # Look for placeholders like {variable_name}
                    matches = re.findall(r'\{(\w+)\}', value)
                    replaced_value = value
                    for match in matches:
                        if match in context:
                            replaced_value = replaced_value.replace(f'{{{match}}}', str(context[match]))
                    new_params[key] = replaced_value
                else:
                    new_params[key] = value
            new_step['parameters'] = new_params

        return new_step

    def create_new_plan(self, task_description: str, context: Dict) -> List[Dict]:
        """Create a new plan using LLM when no template matches"""
        # This would call the LLM to generate a plan
        # For this example, we'll return an empty plan
        # In practice, this would use the LLMCognitivePlanner
        return []
```

### Context-Aware Planning

```python
class ContextAwarePlanner:
    def __init__(self):
        self.context = {
            'time_of_day': 'morning',
            'day_of_week': 'monday',
            'human_activities': [],
            'robot_state': {},
            'environment_state': {},
            'social_context': {},
            'safety_context': {}
        }

    def update_context(self, **kwargs):
        """Update the planning context with new information"""
        for key, value in kwargs.items():
            if key in self.context:
                self.context[key] = value

    def get_context_relevant_prompt(self, task: str) -> str:
        """Generate a context-relevant prompt for the LLM"""
        return f"""
        Generate a plan for: {task}

        CONTEXT:
        - Time of day: {self.context['time_of_day']}
        - Day of week: {self.context['day_of_week']}
        - Human activities in environment: {', '.join(self.context['human_activities'])}
        - Robot state: {self.context['robot_state']}
        - Environment state: {self.context['environment_state']}
        - Social context: {self.context['social_context']}
        - Safety considerations: {self.context['safety_context']}

        Consider the context when generating the plan:
        1. Time-appropriate behavior
        2. Awareness of human activities and privacy
        3. Environmental constraints
        4. Safety requirements
        5. Social norms and expectations

        Output the plan in JSON format:
        """

    def contextual_refinement(self, initial_plan: List[Dict]) -> List[Dict]:
        """Refine plan based on context"""
        refined_plan = []

        for step in initial_plan:
            # Add context-aware modifications to the step
            refined_step = self.refine_step_with_context(step)
            refined_plan.append(refined_step)

        return refined_plan

    def refine_step_with_context(self, step: Dict) -> Dict:
        """Refine a single step based on context"""
        refined_step = step.copy()

        # Example: Adjust navigation speed based on human presence
        if step['action'] == 'navigate':
            if 'near humans' in self.context['human_activities']:
                # Add safety parameters for navigating near humans
                if 'parameters' not in refined_step:
                    refined_step['parameters'] = {}
                refined_step['parameters']['safe_speed'] = True
                refined_step['parameters']['maintain_distance'] = True

        # Example: Adjust communication based on time of day
        elif step['action'] == 'communicate':
            if self.context['time_of_day'] in ['night', 'late']:
                if 'parameters' not in refined_step:
                    refined_step['parameters'] = {}
                refined_step['parameters']['volume'] = 'quiet'

        return refined_step
```

## Safety and Validation

### Plan Validation Framework

```python
class PlanValidator:
    def __init__(self):
        self.safety_rules = [
            self.check_collision_risk,
            self.verify_feasibility,
            self.check_battery_constraints,
            self.validate_social_norms,
            self.ensure_privacy_protection
        ]

    def validate_plan(self, plan: List[Dict], robot_state: Dict) -> Dict:
        """Validate a plan against safety and feasibility constraints"""
        validation_result = {
            'is_valid': True,
            'issues': [],
            'suggestions': [],
            'risk_level': 'low'  # low, medium, high
        }

        for rule in self.safety_rules:
            rule_result = rule(plan, robot_state)
            if not rule_result['valid']:
                validation_result['is_valid'] = False
                validation_result['issues'].extend(rule_result['issues'])
                validation_result['suggestions'].extend(rule_result['suggestions'])

        # Determine overall risk level
        if validation_result['issues']:
            if len(validation_result['issues']) > 5:
                validation_result['risk_level'] = 'high'
            elif len(validation_result['issues']) > 2:
                validation_result['risk_level'] = 'medium'
            else:
                validation_result['risk_level'] = 'medium'

        return validation_result

    def check_collision_risk(self, plan: List[Dict], robot_state: Dict) -> Dict:
        """Check for potential collision risks in the plan"""
        issues = []
        suggestions = []

        for i, step in enumerate(plan):
            if step['action'] == 'navigate':
                # In a real system, this would check navigation maps and predict human movements
                # For this example, we'll just check if navigating to crowded areas
                params = step.get('parameters', {})
                location = params.get('location', '')

                if 'crowded' in location or 'busy' in location:
                    issues.append(f"Step {i+1}: Navigation to {location} may have collision risks")
                    suggestions.append(f"Consider alternative route or slower navigation speed for {location}")

        return {
            'valid': len(issues) == 0,
            'issues': issues,
            'suggestions': suggestions
        }

    def verify_feasibility(self, plan: List[Dict], robot_state: Dict) -> Dict:
        """Verify that the plan is physically feasible"""
        issues = []
        suggestions = []

        for i, step in enumerate(plan):
            action = step['action']
            params = step.get('parameters', {})

            # Check if robot has required capabilities
            if action in ['pick_up', 'place'] and robot_state.get('manipulation_capability', False) is False:
                issues.append(f"Step {i+1}: Robot lacks manipulation capability for {action}")
                suggestions.append("Add manipulation capability or skip manipulation steps")

            # Check if required objects are available
            if action in ['pick_up', 'detect_object'] and 'object' in params:
                obj = params['object']
                if obj not in robot_state.get('detectable_objects', []):
                    issues.append(f"Step {i+1}: Object {obj} may not be detectable by robot")
                    suggestions.append(f"Verify {obj} is in robot's detection range or skip step")

        return {
            'valid': len(issues) == 0,
            'issues': issues,
            'suggestions': suggestions
        }

    def check_battery_constraints(self, plan: List[Dict], robot_state: Dict) -> Dict:
        """Check if the plan respects battery constraints"""
        issues = []
        suggestions = []

        # Estimate battery consumption
        estimated_consumption = self.estimate_battery_usage(plan)
        current_battery = robot_state.get('battery_level', 100.0)
        safety_margin = 20.0  # Keep 20% battery for emergencies

        if current_battery - estimated_consumption < safety_margin:
            issues.append(f"Plan may deplete battery below safety threshold. Current: {current_battery}%, Estimated usage: {estimated_consumption}%")
            suggestions.append("Consider reducing plan scope or recharging before execution")

        return {
            'valid': len(issues) == 0,
            'issues': issues,
            'suggestions': suggestions
        }

    def estimate_battery_usage(self, plan: List[Dict]) -> float:
        """Estimate battery usage for the plan"""
        consumption = 0.0

        for step in plan:
            action = step['action']
            if action == 'navigate':
                # Estimate based on distance (simplified)
                consumption += 2.0  # 2% per navigation action
            elif action in ['pick_up', 'place', 'move_arm']:
                # Manipulation actions
                consumption += 1.0
            elif action == 'communicate':
                # Communication
                consumption += 0.1
            else:
                # Other actions
                consumption += 0.5

        return consumption

    def validate_social_norms(self, plan: List[Dict], robot_state: Dict) -> Dict:
        """Validate plan against social norms and etiquette"""
        issues = []
        suggestions = []

        for i, step in enumerate(plan):
            action = step['action']
            params = step.get('parameters', {})

            # Check for potentially inappropriate actions
            if action == 'navigate' and params.get('location', '').lower() in ['bedroom', 'bathroom']:
                issues.append(f"Step {i+1}: Navigation to private area ({params['location']}) may violate privacy norms")
                suggestions.append(f"Request permission before entering {params['location']} or use alternative approach")

            if action == 'communicate' and params.get('message', '').lower().startswith('hey'):
                # Less formal greeting might be inappropriate in certain contexts
                issues.append(f"Step {i+1}: Informal greeting might not be appropriate")
                suggestions.append("Use more formal greeting like 'Hello' instead of 'Hey'")

        return {
            'valid': len(issues) == 0,
            'issues': issues,
            'suggestions': suggestions
        }

    def ensure_privacy_protection(self, plan: List[Dict], robot_state: Dict) -> Dict:
        """Ensure plan respects privacy requirements"""
        issues = []
        suggestions = []

        for i, step in enumerate(plan):
            action = step['action']
            params = step.get('parameters', {})

            if action in ['navigate', 'detect_object'] and params.get('location', '').lower() in ['bedroom', 'bathroom', 'office']:
                # Check if privacy-sensitive area
                issues.append(f"Step {i+1}: Action in privacy-sensitive area ({params['location']})")
                suggestions.append(f"Implement privacy protection measures when operating in {params['location']}")

        return {
            'valid': len(issues) == 0,
            'issues': issues,
            'suggestions': suggestions
        }
```

## Performance Optimization and Evaluation

### Planning Performance Metrics

```python
import time
import statistics
from collections import defaultdict, deque

class PlanningPerformanceEvaluator:
    def __init__(self):
        self.metrics = {
            'planning_time': deque(maxlen=100),
            'plan_quality_scores': deque(maxlen=100),
            'success_rate': 0.0,
            'average_plan_steps': deque(maxlen=100),
            'context_accuracy': deque(maxlen=100),
            'safety_violations': 0,
            'total_plans_generated': 0,
            'total_successful_executions': 0
        }

    def record_planning_event(self, task_complexity: str, planning_time: float,
                            plan_quality: float, num_steps: int):
        """Record a planning event for metrics tracking"""
        self.metrics['planning_time'].append(planning_time)
        self.metrics['plan_quality_scores'].append(plan_quality)
        self.metrics['average_plan_steps'].append(num_steps)
        self.metrics['total_plans_generated'] += 1

    def record_execution_result(self, success: bool, context_accuracy: float = None):
        """Record plan execution result"""
        if success:
            self.metrics['total_successful_executions'] += 1

        if context_accuracy is not None:
            self.metrics['context_accuracy'].append(context_accuracy)

    def record_safety_violation(self):
        """Record a safety violation"""
        self.metrics['safety_violations'] += 1

    def get_current_metrics(self) -> Dict:
        """Get current performance metrics"""
        planning_times = list(self.metrics['planning_time'])
        quality_scores = list(self.metrics['plan_quality_scores'])
        context_accuracies = list(self.metrics['context_accuracy'])
        plan_steps = list(self.metrics['average_plan_steps'])

        total_plans = self.metrics['total_plans_generated']
        successful_executions = self.metrics['total_successful_executions']

        metrics_summary = {
            'avg_planning_time': statistics.mean(planning_times) if planning_times else 0,
            'std_planning_time': statistics.stdev(planning_times) if len(planning_times) > 1 else 0,
            'min_planning_time': min(planning_times) if planning_times else 0,
            'max_planning_time': max(planning_times) if planning_times else 0,

            'avg_plan_quality': statistics.mean(quality_scores) if quality_scores else 0,
            'std_plan_quality': statistics.stdev(quality_scores) if len(quality_scores) > 1 else 0,

            'avg_context_accuracy': statistics.mean(context_accuracies) if context_accuracies else 0,

            'avg_plan_steps': statistics.mean(plan_steps) if plan_steps else 0,

            'success_rate': successful_executions / total_plans if total_plans > 0 else 0,

            'safety_violations': self.metrics['safety_violations'],
            'total_plans_generated': total_plans,
            'total_successful_executions': successful_executions
        }

        return metrics_summary

    def print_metrics_report(self):
        """Print a formatted metrics report"""
        metrics = self.get_current_metrics()

        print("LLM-Based Cognitive Planning Performance Report:")
        print("=" * 50)
        print(f"Total Plans Generated: {metrics['total_plans_generated']}")
        print(f"Successful Executions: {metrics['total_successful_executions']}")
        print(f"Success Rate: {metrics['success_rate']:.2%}")
        print(f"Safety Violations: {metrics['safety_violations']}")
        print()
        print("Planning Performance:")
        print(f"  Avg Planning Time: {metrics['avg_planning_time']:.3f}s")
        print(f"  Std Planning Time: {metrics['std_planning_time']:.3f}s")
        print(f"  Min Planning Time: {metrics['min_planning_time']:.3f}s")
        print(f"  Max Planning Time: {metrics['max_planning_time']:.3f}s")
        print()
        print("Plan Quality:")
        print(f"  Avg Quality Score: {metrics['avg_plan_quality']:.3f}")
        print(f"  Std Quality Score: {metrics['std_plan_quality']:.3f}")
        print(f"  Avg Context Accuracy: {metrics['avg_context_accuracy']:.3f}")
        print(f"  Avg Plan Steps: {metrics['avg_plan_steps']:.1f}")
```

## Troubleshooting and Debugging

### Planning System Diagnostics

```python
class PlanningDiagnostics:
    def __init__(self, node):
        self.node = node
        self.diagnostics = {
            'llm_connection': {'status': 'unknown', 'details': ''},
            'context_awareness': {'status': 'unknown', 'details': ''},
            'safety_validation': {'status': 'unknown', 'details': ''},
            'execution_monitoring': {'status': 'unknown', 'details': ''}
        }

    def run_comprehensive_diagnostics(self):
        """Run comprehensive diagnostics on the planning system"""
        self.check_llm_connection()
        self.check_context_awareness()
        self.check_safety_validation()
        self.check_execution_monitoring()

        return self.diagnostics

    def check_llm_connection(self):
        """Check LLM connection and accessibility"""
        try:
            # Test LLM API connection
            import openai
            response = openai.ChatCompletion.create(
                model="gpt-3.5-turbo",
                messages=[{"role": "user", "content": "test"}],
                max_tokens=5
            )

            self.diagnostics['llm_connection']['status'] = 'ok'
            self.diagnostics['llm_connection']['details'] = 'LLM API accessible'
        except Exception as e:
            self.diagnostics['llm_connection']['status'] = 'error'
            self.diagnostics['llm_connection']['details'] = f'LLM connection failed: {str(e)}'

    def check_context_awareness(self):
        """Check if context system is working properly"""
        try:
            # Check if context can be updated and retrieved
            context_system = getattr(self.node, 'context_aware_planner', None)
            if context_system:
                # Test context update
                context_system.update_context(test_value='diagnostic')
                current_context = context_system.context.get('test_value')

                if current_context == 'diagnostic':
                    self.diagnostics['context_awareness']['status'] = 'ok'
                    self.diagnostics['context_awareness']['details'] = 'Context system functional'
                else:
                    self.diagnostics['context_awareness']['status'] = 'warning'
                    self.diagnostics['context_awareness']['details'] = 'Context update not working properly'
            else:
                self.diagnostics['context_awareness']['status'] = 'warning'
                self.diagnostics['context_awareness']['details'] = 'Context system not initialized'
        except Exception as e:
            self.diagnostics['context_awareness']['status'] = 'error'
            self.diagnostics['context_awareness']['details'] = f'Context system error: {str(e)}'

    def check_safety_validation(self):
        """Check safety validation system"""
        try:
            # Check if safety validator exists and works
            safety_validator = getattr(self.node, 'plan_validator', None)
            if safety_validator:
                # Test with a simple plan
                test_plan = [{'action': 'navigate', 'description': 'move forward'}]
                result = safety_validator.validate_plan(test_plan, {})

                if isinstance(result, dict):
                    self.diagnostics['safety_validation']['status'] = 'ok'
                    self.diagnostics['safety_validation']['details'] = 'Safety validation functional'
                else:
                    self.diagnostics['safety_validation']['status'] = 'error'
                    self.diagnostics['safety_validation']['details'] = 'Safety validation not returning expected format'
            else:
                self.diagnostics['safety_validation']['status'] = 'warning'
                self.diagnostics['safety_validation']['details'] = 'Safety validator not initialized'
        except Exception as e:
            self.diagnostics['safety_validation']['status'] = 'error'
            self.diagnostics['safety_validation']['details'] = f'Safety validation error: {str(e)}'

    def check_execution_monitoring(self):
        """Check execution monitoring system"""
        try:
            # Check if execution monitor exists
            executor = getattr(self.node, 'plan_executor', None)
            if executor:
                # Check if required publishers exist
                if hasattr(executor, 'status_pub') and executor.status_pub:
                    self.diagnostics['execution_monitoring']['status'] = 'ok'
                    self.diagnostics['execution_monitoring']['details'] = 'Execution monitoring functional'
                else:
                    self.diagnostics['execution_monitoring']['status'] = 'error'
                    self.diagnostics['execution_monitoring']['details'] = 'Execution monitoring publishers not available'
            else:
                self.diagnostics['execution_monitoring']['status'] = 'warning'
                self.diagnostics['execution_monitoring']['details'] = 'Executor not initialized'
        except Exception as e:
            self.diagnostics['execution_monitoring']['status'] = 'error'
            self.diagnostics['execution_monitoring']['details'] = f'Execution monitoring error: {str(e)}'

    def get_system_health(self) -> str:
        """Get overall system health"""
        statuses = [self.diagnostics[key]['status'] for key in self.diagnostics]

        if 'error' in statuses:
            return 'error'
        elif 'warning' in statuses:
            return 'warning'
        else:
            return 'ok'
```

## Best Practices for LLM-Based Planning

### 1. Safety-First Approach

- Always validate LLM-generated plans before execution
- Implement multiple safety checks and constraints
- Use conservative parameters by default
- Plan for graceful degradation and error recovery

### 2. Context Management

- Maintain accurate and up-to-date robot state
- Consider temporal and social context
- Update context continuously during execution
- Handle context uncertainty appropriately

### 3. Performance Optimization

- Cache frequently used plans and patterns
- Use appropriate LLM models for the task complexity
- Implement plan refinement and optimization
- Monitor and optimize planning latency

## Exercises

1. **LLM Integration Exercise**: Implement an LLM-based planner that can interpret natural language commands and generate executable robot plans.

2. **Safety Validation Exercise**: Create a comprehensive safety validation system that checks LLM-generated plans for potential risks and constraints.

3. **Context-Aware Planning Exercise**: Develop a context-aware planning system that adapts robot behavior based on environmental and social context.

## Summary

This chapter covered LLM-based cognitive planning for humanoid robotics, including LLM architectures, prompt engineering, ROS 2 integration, advanced planning techniques, and safety validation. We explored how LLMs can enhance robot autonomy by enabling natural language interaction and sophisticated task decomposition. Proper implementation of LLM-based planning requires careful attention to safety, validation, and context awareness to ensure reliable and safe robot behavior.

## Further Reading

- [Large Language Models for Robotics](https://arxiv.org/abs/2309.17176)
- [Prompt Engineering for Robotics](https://arxiv.org/abs/2305.15771)
- [Safe Robot Planning with LLMs](https://ieeexplore.ieee.org/document/9817245)
- [Human-Robot Interaction with Natural Language](https://dl.acm.org/doi/10.1145/3411764.3445522)

---

**Previous Chapter**: [Voice-to-Action](./voice-to-action.md)
**Next Chapter**: [Capstone: Autonomous Humanoid](./capstone-autonomous-humanoid.md)