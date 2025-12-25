---
sidebar_position: 1
title: "Voice-to-Action"
---

# Voice-to-Action for Humanoid Robotics

## Learning Objectives

By the end of this chapter, you will be able to:
- Understand the architecture of voice-to-action systems for humanoid robots
- Implement speech recognition and natural language processing for robot commands
- Design voice command grammars suitable for humanoid robot control
- Integrate voice processing with ROS 2 for real-time robot control
- Create robust voice command validation and error handling systems
- Deploy voice-to-action systems on humanoid robots with appropriate safety measures

## Introduction to Voice-to-Action Systems

Voice-to-Action systems enable humanoid robots to understand and execute commands given through natural language voice input. This technology represents a crucial interface between humans and robots, allowing for intuitive interaction without requiring specialized knowledge of robot control interfaces.

### Key Components of Voice-to-Action Systems

1. **Speech Recognition**: Converting audio input to text
2. **Natural Language Understanding (NLU)**: Interpreting the meaning of commands
3. **Action Mapping**: Translating understood commands to robot actions
4. **Execution Layer**: Executing mapped actions on the robot
5. **Feedback System**: Providing confirmation and status updates

### Benefits of Voice Control for Humanoid Robots

- **Intuitive Interaction**: Natural communication method for humans
- **Accessibility**: Enables interaction for users with mobility limitations
- **Hands-Free Operation**: Allows users to control robots while performing other tasks
- **Social Integration**: Facilitates more natural human-robot interaction
- **Rapid Prototyping**: Quick development of robot behaviors through voice commands

## Speech Recognition Technologies

### Overview of Speech Recognition

Speech recognition is the process of converting spoken language into text. For humanoid robotics, this involves real-time processing of audio input to understand user commands.

### Common Speech Recognition Approaches

1. **Cloud-Based Recognition**: Services like Google Cloud Speech-to-Text, AWS Transcribe
2. **On-Device Recognition**: Libraries like CMU Sphinx, Kaldi, or Vosk
3. **Hybrid Approaches**: Combination of on-device and cloud processing

### Speech Recognition with Vosk

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from sensor_msgs.msg import AudioData
import vosk
import json
import pyaudio
import wave
import threading

class VoiceToActionNode(Node):
    def __init__(self):
        super().__init__('voice_to_action_node')

        # Initialize Vosk speech recognition model
        # You need to download a model from https://alphacephei.com/vosk/models
        self.model = vosk.Model("path/to/vosk-model")  # Replace with actual model path
        self.rec = vosk.KaldiRecognizer(self.model, 16000)

        # Publishers and subscribers
        self.command_publisher = self.create_publisher(String, 'voice_commands', 10)
        self.audio_subscriber = self.create_subscription(
            AudioData, 'audio_input', self.audio_callback, 10
        )

        # Voice command processing
        self.audio_queue = []
        self.command_buffer = ""
        self.listening = True

        # Start audio processing thread
        self.audio_thread = threading.Thread(target=self.process_audio_stream)
        self.audio_thread.start()

    def audio_callback(self, msg):
        """Process incoming audio data"""
        # Add audio data to processing queue
        self.audio_queue.append(msg.data)

    def process_audio_stream(self):
        """Process audio stream for speech recognition"""
        while self.listening:
            if self.audio_queue:
                audio_data = self.audio_queue.pop(0)

                # Process audio chunk with Vosk
                if self.rec.AcceptWaveform(audio_data):
                    result = self.rec.Result()
                    result_dict = json.loads(result)

                    if 'text' in result_dict and result_dict['text']:
                        self.process_recognized_command(result_dict['text'])
                else:
                    # Partial result (interim recognition)
                    partial_result = self.rec.PartialResult()
                    partial_dict = json.loads(partial_result)
                    if 'partial' in partial_dict:
                        self.command_buffer = partial_dict['partial']

    def process_recognized_command(self, command_text):
        """Process the recognized voice command"""
        self.get_logger().info(f'Recognized command: {command_text}')

        # Clean and validate the command
        cleaned_command = self.clean_command(command_text.lower().strip())

        if cleaned_command:
            # Publish the command to be processed by action mapping
            cmd_msg = String()
            cmd_msg.data = cleaned_command
            self.command_publisher.publish(cmd_msg)

            # Process the command immediately
            self.execute_voice_command(cleaned_command)

    def clean_command(self, command):
        """Clean and normalize the voice command"""
        # Remove common filler words and normalize
        fillers = ['um', 'uh', 'like', 'you know', 'so']
        for filler in fillers:
            command = command.replace(filler, '')

        # Remove extra whitespace
        command = ' '.join(command.split())

        return command if command else ""

    def execute_voice_command(self, command):
        """Execute voice command by mapping to robot actions"""
        # Define command patterns and their corresponding actions
        command_patterns = {
            'move forward': self.move_forward,
            'move backward': self.move_backward,
            'turn left': self.turn_left,
            'turn right': self.turn_right,
            'stop': self.stop_robot,
            'raise left arm': self.raise_left_arm,
            'raise right arm': self.raise_right_arm,
            'lower arms': self.lower_arms,
            'walk to kitchen': self.navigate_to_kitchen,
            'bring me water': self.fetch_water,
        }

        # Find matching command
        for pattern, action_func in command_patterns.items():
            if pattern in command:
                self.get_logger().info(f'Executing command: {pattern}')
                action_func()
                return True

        # If no exact match, try fuzzy matching
        return self.fuzzy_match_command(command)

    def fuzzy_match_command(self, command):
        """Perform fuzzy matching for commands that don't match exactly"""
        import difflib

        # Define possible commands
        possible_commands = [
            'move forward', 'move backward', 'turn left', 'turn right',
            'stop', 'raise left arm', 'raise right arm', 'lower arms',
            'walk to kitchen', 'bring me water'
        ]

        # Find the best match
        matches = difflib.get_close_matches(command, possible_commands, n=1, cutoff=0.6)

        if matches:
            best_match = matches[0]
            self.get_logger().info(f'Fuzzy matched command: {best_match}')

            # Execute the matched command
            command_patterns = {
                'move forward': self.move_forward,
                'move backward': self.move_backward,
                'turn left': self.turn_left,
                'turn right': self.turn_right,
                'stop': self.stop_robot,
                'raise left arm': self.raise_left_arm,
                'raise right arm': self.raise_right_arm,
                'lower arms': self.lower_arms,
                'walk to kitchen': self.navigate_to_kitchen,
                'bring me water': self.fetch_water,
            }

            if best_match in command_patterns:
                command_patterns[best_match]()
                return True

        self.get_logger().warn(f'Unrecognized command: {command}')
        return False

    # Action implementations
    def move_forward(self):
        """Move robot forward"""
        self.get_logger().info('Moving forward')
        # Implementation would send appropriate messages to robot controller

    def move_backward(self):
        """Move robot backward"""
        self.get_logger().info('Moving backward')
        # Implementation would send appropriate messages to robot controller

    def turn_left(self):
        """Turn robot left"""
        self.get_logger().info('Turning left')
        # Implementation would send appropriate messages to robot controller

    def turn_right(self):
        """Turn robot right"""
        self.get_logger().info('Turning right')
        # Implementation would send appropriate messages to robot controller

    def stop_robot(self):
        """Stop robot movement"""
        self.get_logger().info('Stopping robot')
        # Implementation would send stop command to robot controller

    def raise_left_arm(self):
        """Raise left arm"""
        self.get_logger().info('Raising left arm')
        # Implementation would send appropriate joint commands

    def raise_right_arm(self):
        """Raise right arm"""
        self.get_logger().info('Raising right arm')
        # Implementation would send appropriate joint commands

    def lower_arms(self):
        """Lower both arms"""
        self.get_logger().info('Lowering arms')
        # Implementation would send appropriate joint commands

    def navigate_to_kitchen(self):
        """Navigate to kitchen location"""
        self.get_logger().info('Navigating to kitchen')
        # Implementation would send navigation goal to Nav2

    def fetch_water(self):
        """Fetch water task"""
        self.get_logger().info('Fetching water')
        # Implementation would execute complex manipulation task

def main(args=None):
    rclpy.init(args=args)
    voice_node = VoiceToActionNode()

    try:
        rclpy.spin(voice_node)
    except KeyboardInterrupt:
        voice_node.listening = False
        voice_node.audio_thread.join()
        pass
    finally:
        voice_node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Natural Language Processing for Robot Commands

### Intent Recognition

Intent recognition is crucial for understanding what the user wants the robot to do:

```python
import re
from typing import Dict, List, Tuple

class IntentRecognizer:
    def __init__(self):
        # Define intent patterns
        self.intent_patterns = {
            'move': [
                r'move\s+(forward|backward|left|right)',
                r'go\s+(forward|backward|left|right)',
                r'walk\s+(forward|backward|left|right)',
                r'step\s+(forward|backward|left|right)',
            ],
            'turn': [
                r'turn\s+(left|right)',
                r'rotate\s+(left|right)',
                r'pivot\s+(left|right)',
            ],
            'navigation': [
                r'go to (.*?)(?:\s|$)',
                r'walk to (.*?)(?:\s|$)',
                r'navigate to (.*?)(?:\s|$)',
                r'move to (.*?)(?:\s|$)',
            ],
            'manipulation': [
                r'pick up (.*?)(?:\s|$)',
                r'grab (.*?)(?:\s|$)',
                r'lift (.*?)(?:\s|$)',
                r'pick (.*?)(?:\s|$)',
            ],
            'stop': [
                r'stop',
                r'hold',
                r'wait',
                r'pause',
            ],
            'greeting': [
                r'hello',
                r'hi',
                r'hey',
                r'good morning',
                r'good afternoon',
                r'good evening',
            ]
        }

    def recognize_intent(self, text: str) -> Tuple[str, Dict]:
        """Recognize intent and extract entities from text"""
        text_lower = text.lower()

        for intent, patterns in self.intent_patterns.items():
            for pattern in patterns:
                match = re.search(pattern, text_lower)
                if match:
                    entities = match.groups()
                    return intent, {'entities': entities, 'text': text}

        return 'unknown', {'text': text}

    def extract_location(self, text: str) -> str:
        """Extract location from navigation commands"""
        # Common locations in a home environment
        locations = [
            'kitchen', 'living room', 'bedroom', 'bathroom', 'office',
            'dining room', 'hallway', 'garage', 'garden', 'entrance'
        ]

        text_lower = text.lower()
        for location in locations:
            if location in text_lower:
                return location

        return None

    def extract_object(self, text: str) -> str:
        """Extract object from manipulation commands"""
        # Common objects a robot might manipulate
        objects = [
            'water', 'cup', 'bottle', 'book', 'phone', 'keys',
            'food', 'medicine', 'toy', 'clothes', 'trash'
        ]

        text_lower = text.lower()
        for obj in objects:
            if obj in text_lower:
                return obj

        return None
```

### Command Validation and Safety

```python
class CommandValidator:
    def __init__(self):
        self.safety_keywords = ['emergency', 'stop', 'help', 'danger']
        self.prohibited_actions = ['shoot', 'kill', 'hurt', 'break', 'destroy']

        # Define safe movement parameters for humanoid robot
        self.max_linear_velocity = 0.5  # m/s
        self.max_angular_velocity = 0.5  # rad/s
        self.max_lift_height = 1.8  # meters (above ground)

    def validate_command(self, command: str, context: Dict) -> Tuple[bool, str]:
        """Validate command for safety and feasibility"""

        # Check for prohibited actions
        for prohibited in self.prohibited_actions:
            if prohibited in command.lower():
                return False, f"Command contains prohibited action: {prohibited}"

        # Check for safety keywords that require immediate attention
        for keyword in self.safety_keywords:
            if keyword in command.lower():
                return True, f"Safety keyword detected: {keyword}"

        # Validate navigation commands
        if any(nav_cmd in command.lower() for nav_cmd in ['go to', 'navigate to', 'walk to']):
            location = self.extract_location(command)
            if not location:
                return False, "No valid location specified in navigation command"

            # Check if location is in known map
            if not self.is_known_location(location, context):
                return False, f"Unknown location: {location}"

        # Validate manipulation commands
        if any(manip_cmd in command.lower() for manip_cmd in ['pick up', 'grab', 'lift']):
            obj = self.extract_object(command)
            if not obj:
                return False, "No valid object specified in manipulation command"

            # Check if object is reachable
            if not self.is_object_reachable(obj, context):
                return False, f"Object {obj} is not reachable"

        return True, "Command is valid"

    def is_known_location(self, location: str, context: Dict) -> bool:
        """Check if location exists in robot's map"""
        # This would check against the robot's known locations
        # Implementation would depend on navigation system
        known_locations = context.get('known_locations', [])
        return location.lower() in [loc.lower() for loc in known_locations]

    def is_object_reachable(self, obj: str, context: Dict) -> bool:
        """Check if object is reachable by robot"""
        # This would check against object detection and robot kinematics
        # Implementation would depend on perception and manipulation systems
        return True  # Placeholder
```

## Voice Command Grammar Design

### Context-Free Grammar for Robot Commands

```python
class VoiceCommandGrammar:
    def __init__(self):
        # Define grammar rules for voice commands
        self.grammar_rules = {
            'S': [['MOVE', 'DIRECTION'], ['NAVIGATE', 'LOCATION'], ['MANIPULATE', 'OBJECT'], ['STOP']],
            'MOVE': ['move', 'go', 'walk', 'step'],
            'DIRECTION': ['forward', 'backward', 'left', 'right'],
            'NAVIGATE': ['go to', 'navigate to', 'walk to', 'move to'],
            'LOCATION': ['kitchen', 'living room', 'bedroom', 'bathroom', 'office'],
            'MANIPULATE': ['pick up', 'grab', 'lift', 'take'],
            'OBJECT': ['water', 'cup', 'bottle', 'book', 'phone'],
            'STOP': ['stop', 'halt', 'wait', 'pause']
        }

    def generate_possible_commands(self) -> List[str]:
        """Generate possible voice commands based on grammar"""
        commands = []

        # Generate move commands
        for move in self.grammar_rules['MOVE']:
            for direction in self.grammar_rules['DIRECTION']:
                commands.append(f"{move} {direction}")

        # Generate navigation commands
        for nav in self.grammar_rules['NAVIGATE']:
            for loc in self.grammar_rules['LOCATION']:
                commands.append(f"{nav} {loc}")

        # Generate manipulation commands
        for manip in self.grammar_rules['MANIPULATE']:
            for obj in self.grammar_rules['OBJECT']:
                commands.append(f"{manip} {obj}")

        # Add stop commands
        for stop in self.grammar_rules['STOP']:
            commands.append(stop)

        return commands

    def validate_command_structure(self, command: str) -> bool:
        """Validate command structure against grammar"""
        words = command.lower().split()

        # Check against known patterns
        if len(words) >= 2:
            # Check move commands: move + direction
            if words[0] in self.grammar_rules['MOVE'] and words[1] in self.grammar_rules['DIRECTION']:
                return True

            # Check navigate commands: navigate + location (multi-word locations handled separately)
            if words[0] in ['go', 'navigate', 'walk', 'move'] and words[1] == 'to':
                location = ' '.join(words[2:])
                # Check if location matches any known location pattern
                for loc in self.grammar_rules['LOCATION']:
                    if loc in location:
                        return True

        return False
```

## ROS 2 Integration for Voice Commands

### Voice Command Processing Pipeline

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from geometry_msgs.msg import Twist
from action_msgs.msg import GoalStatus
from rclpy.action import ActionClient
from nav2_msgs.action import NavigateToPose
from geometry_msgs.msg import PoseStamped
import json

class VoiceCommandProcessor(Node):
    def __init__(self):
        super().__init__('voice_command_processor')

        # Publishers for robot control
        self.cmd_vel_pub = self.create_publisher(Twist, '/cmd_vel', 10)

        # Navigation action client
        self.nav_client = ActionClient(self, NavigateToPose, 'navigate_to_pose')

        # Subscriber for voice commands
        self.voice_sub = self.create_subscription(
            String, 'voice_commands', self.voice_command_callback, 10
        )

        # Feedback publisher
        self.feedback_pub = self.create_publisher(String, 'voice_feedback', 10)

        # Initialize NLP components
        self.intent_recognizer = IntentRecognizer()
        self.command_validator = CommandValidator()
        self.grammar = VoiceCommandGrammar()

        # Robot state
        self.robot_state = {
            'location': 'unknown',
            'battery_level': 100.0,
            'current_task': None,
            'known_locations': ['kitchen', 'living room', 'bedroom', 'bathroom']
        }

    def voice_command_callback(self, msg):
        """Process incoming voice command"""
        command = msg.data
        self.get_logger().info(f'Processing voice command: {command}')

        # Validate command
        is_valid, validation_msg = self.command_validator.validate_command(
            command, self.robot_state
        )

        if not is_valid:
            self.send_feedback(f"Command rejected: {validation_msg}")
            return

        # Recognize intent
        intent, entities = self.intent_recognizer.recognize_intent(command)

        # Execute based on intent
        success = self.execute_intent(intent, entities, command)

        if success:
            self.send_feedback(f"Command executed: {command}")
        else:
            self.send_feedback(f"Command failed: {command}")

    def execute_intent(self, intent: str, entities: Dict, original_command: str) -> bool:
        """Execute action based on recognized intent"""
        try:
            if intent == 'move':
                return self.execute_move_command(entities)
            elif intent == 'turn':
                return self.execute_turn_command(entities)
            elif intent == 'navigation':
                return self.execute_navigation_command(entities)
            elif intent == 'manipulation':
                return self.execute_manipulation_command(entities)
            elif intent == 'stop':
                return self.execute_stop_command()
            elif intent == 'greeting':
                return self.execute_greeting_command(entities)
            else:
                self.get_logger().warn(f'Unknown intent: {intent}')
                return False
        except Exception as e:
            self.get_logger().error(f'Error executing intent {intent}: {str(e)}')
            return False

    def execute_move_command(self, entities: Dict) -> bool:
        """Execute move command"""
        if 'entities' in entities and entities['entities']:
            direction = entities['entities'][0]

            twist = Twist()
            if direction == 'forward':
                twist.linear.x = 0.3  # m/s
            elif direction == 'backward':
                twist.linear.x = -0.3
            elif direction == 'left':
                twist.linear.y = 0.3
            elif direction == 'right':
                twist.linear.y = -0.3

            self.cmd_vel_pub.publish(twist)
            self.get_logger().info(f'Moving {direction}')
            return True

        return False

    def execute_turn_command(self, entities: Dict) -> bool:
        """Execute turn command"""
        if 'entities' in entities and entities['entities']:
            direction = entities['entities'][0]

            twist = Twist()
            if direction == 'left':
                twist.angular.z = 0.5  # rad/s
            elif direction == 'right':
                twist.angular.z = -0.5

            self.cmd_vel_pub.publish(twist)
            self.get_logger().info(f'Turning {direction}')
            return True

        return False

    def execute_navigation_command(self, entities: Dict) -> bool:
        """Execute navigation command"""
        if 'text' in entities:
            location = self.intent_recognizer.extract_location(entities['text'])
            if location:
                # Send navigation goal
                goal_msg = NavigateToPose.Goal()
                goal_msg.pose.header.frame_id = 'map'

                # This would be replaced with actual coordinates for the location
                # In a real system, you'd have a map of locations with coordinates
                if location == 'kitchen':
                    goal_msg.pose.pose.position.x = 2.0
                    goal_msg.pose.pose.position.y = 1.0
                elif location == 'living room':
                    goal_msg.pose.pose.position.x = 0.0
                    goal_msg.pose.pose.position.y = 0.0
                # Add more locations as needed

                goal_msg.pose.pose.orientation.w = 1.0  # Default orientation

                # Wait for action server
                self.nav_client.wait_for_server()

                # Send goal
                future = self.nav_client.send_goal_async(goal_msg)
                future.add_done_callback(self.navigation_goal_callback)

                self.get_logger().info(f'Navigating to {location}')
                return True

        return False

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
            self.send_feedback('Navigation succeeded')
        else:
            self.get_logger().info(f'Navigation failed with status: {status}')
            self.send_feedback('Navigation failed')

    def execute_manipulation_command(self, entities: Dict) -> bool:
        """Execute manipulation command"""
        if 'text' in entities:
            obj = self.intent_recognizer.extract_object(entities['text'])
            if obj:
                # This would trigger manipulation behavior
                # In a real system, this would involve complex manipulation planning
                self.get_logger().info(f'Trying to manipulate {obj}')

                # Placeholder for manipulation logic
                # This would involve perception, planning, and execution
                return True

        return False

    def execute_stop_command(self) -> bool:
        """Execute stop command"""
        # Stop all robot movement
        stop_msg = Twist()
        self.cmd_vel_pub.publish(stop_msg)

        self.get_logger().info('Stopping robot')
        return True

    def execute_greeting_command(self, entities: Dict) -> bool:
        """Execute greeting command"""
        greeting = entities.get('text', 'Hello').lower()

        responses = {
            'hello': 'Hello! How can I assist you today?',
            'hi': 'Hi there! What can I do for you?',
            'hey': 'Hey! Ready to help!',
            'good morning': 'Good morning! How are you today?',
            'good afternoon': 'Good afternoon! What can I help with?',
            'good evening': 'Good evening! How can I assist you?'
        }

        for greeting_key, response in responses.items():
            if greeting_key in greeting:
                self.send_feedback(response)
                return True

        # Default response
        self.send_feedback('Hello! How can I help you?')
        return True

    def send_feedback(self, message: str):
        """Send feedback to user"""
        feedback_msg = String()
        feedback_msg.data = message
        self.feedback_pub.publish(feedback_msg)
        self.get_logger().info(f'Feedback: {message}')

def main(args=None):
    rclpy.init(args=args)
    processor = VoiceCommandProcessor()

    try:
        rclpy.spin(processor)
    except KeyboardInterrupt:
        pass
    finally:
        processor.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## Advanced Voice Processing Techniques

### Wake Word Detection

```python
import numpy as np
import pyaudio
import threading
import time

class WakeWordDetector:
    def __init__(self, wake_words=['robot', 'hey robot', 'assistant']):
        self.wake_words = wake_words
        self.detected = False
        self.listening = False

        # Audio parameters
        self.format = pyaudio.paInt16
        self.channels = 1
        self.rate = 16000
        self.chunk = 1024
        self.audio = pyaudio.PyAudio()

        # Initialize VAD (Voice Activity Detection) - simplified approach
        self.energy_threshold = 1000  # Adjust based on environment
        self.silence_threshold = 500
        self.silence_duration = 0.5  # seconds of silence to consider end of speech

    def start_listening(self):
        """Start wake word detection"""
        self.listening = True
        stream = self.audio.open(
            format=self.format,
            channels=self.channels,
            rate=self.rate,
            input=True,
            frames_per_buffer=self.chunk
        )

        print("Listening for wake word...")

        while self.listening:
            data = stream.read(self.chunk)
            audio_data = np.frombuffer(data, dtype=np.int16)

            # Calculate energy of audio chunk
            energy = np.sum(audio_data ** 2) / len(audio_data)

            if energy > self.energy_threshold:
                # Potential speech detected, listen for wake word
                self.listen_for_command(stream)

        stream.stop_stream()
        stream.close()

    def listen_for_command(self, stream):
        """Listen for command after wake word"""
        # In a real implementation, this would use speech recognition
        # to detect if the wake word was actually spoken
        # For this example, we'll just return True to indicate wake word detected

        print("Wake word detected! Listening for command...")
        self.detected = True

        # Record command (simplified - in reality would record until silence)
        command_frames = []
        silence_count = 0
        max_silence_frames = int(self.silence_duration * self.rate / self.chunk)

        for _ in range(50):  # Record for a limited time
            data = stream.read(self.chunk)
            audio_data = np.frombuffer(data, dtype=np.int16)
            command_frames.append(data)

            energy = np.sum(audio_data ** 2) / len(audio_data)
            if energy < self.silence_threshold:
                silence_count += 1
                if silence_count > max_silence_frames:
                    break
            else:
                silence_count = 0

        # Process the recorded command
        if command_frames:
            # In a real system, this would be sent to speech recognition
            print("Command recorded, processing...")

        # Reset for next detection
        time.sleep(1)  # Brief pause before listening again
        self.detected = False

    def stop_listening(self):
        """Stop wake word detection"""
        self.listening = False
```

### Voice Command Context Management

```python
from datetime import datetime, timedelta
from typing import Optional

class VoiceContextManager:
    def __init__(self):
        self.context = {
            'current_task': None,
            'task_start_time': None,
            'user_preferences': {},
            'recent_commands': [],
            'conversation_history': [],
            'attention_state': 'idle'  # idle, listening, processing, executing
        }
        self.command_history_limit = 10
        self.conversation_history_limit = 20

    def update_context(self, **kwargs):
        """Update context with new information"""
        for key, value in kwargs.items():
            self.context[key] = value

    def add_command_to_history(self, command: str):
        """Add command to history"""
        self.context['recent_commands'].append({
            'command': command,
            'timestamp': datetime.now()
        })

        # Limit history size
        if len(self.context['recent_commands']) > self.command_history_limit:
            self.context['recent_commands'] = self.context['recent_commands'][-self.command_history_limit:]

    def get_recent_commands(self, limit: int = 5) -> list:
        """Get recent commands"""
        return self.context['recent_commands'][-limit:]

    def start_task(self, task_name: str):
        """Start a new task and update context"""
        self.context['current_task'] = task_name
        self.context['task_start_time'] = datetime.now()
        self.context['attention_state'] = 'executing'

    def complete_task(self):
        """Complete current task"""
        if self.context['current_task']:
            task_duration = datetime.now() - self.context['task_start_time']
            self.context['conversation_history'].append({
                'task': self.context['current_task'],
                'duration': task_duration,
                'completed_at': datetime.now()
            })

            self.context['current_task'] = None
            self.context['task_start_time'] = None
            self.context['attention_state'] = 'idle'

    def is_task_active(self) -> bool:
        """Check if a task is currently active"""
        return self.context['current_task'] is not None

    def get_active_task(self) -> Optional[str]:
        """Get the currently active task"""
        return self.context['current_task']

    def set_attention_state(self, state: str):
        """Set attention state"""
        valid_states = ['idle', 'listening', 'processing', 'executing']
        if state in valid_states:
            self.context['attention_state'] = state

    def get_attention_state(self) -> str:
        """Get current attention state"""
        return self.context['attention_state']

    def get_context_summary(self) -> dict:
        """Get a summary of the current context"""
        return {
            'attention_state': self.context['attention_state'],
            'current_task': self.context['current_task'],
            'recent_commands_count': len(self.context['recent_commands']),
            'conversation_history_count': len(self.context['conversation_history'])
        }
```

## Performance Optimization and Evaluation

### Voice Command Performance Metrics

```python
import time
from collections import deque
import statistics

class VoiceCommandPerformanceEvaluator:
    def __init__(self):
        self.response_times = deque(maxlen=100)  # Keep last 100 measurements
        self.success_count = 0
        self.failure_count = 0
        self.total_commands = 0
        self.start_time = None

    def start_command_processing(self):
        """Start timing command processing"""
        self.start_time = time.time()

    def end_command_processing(self, success: bool):
        """End timing command processing and record results"""
        if self.start_time:
            response_time = time.time() - self.start_time
            self.response_times.append(response_time)

            if success:
                self.success_count += 1
            else:
                self.failure_count += 1

            self.total_commands += 1
            self.start_time = None

    def get_metrics(self) -> dict:
        """Get current performance metrics"""
        if not self.response_times:
            return {
                'avg_response_time': 0.0,
                'min_response_time': 0.0,
                'max_response_time': 0.0,
                'std_response_time': 0.0,
                'success_rate': 0.0,
                'total_commands': 0
            }

        response_times_list = list(self.response_times)
        total_attempts = self.success_count + self.failure_count

        metrics = {
            'avg_response_time': statistics.mean(response_times_list),
            'min_response_time': min(response_times_list),
            'max_response_time': max(response_times_list),
            'std_response_time': statistics.stdev(response_times_list) if len(response_times_list) > 1 else 0.0,
            'success_rate': self.success_count / total_attempts if total_attempts > 0 else 0.0,
            'total_commands': self.total_commands
        }

        return metrics

    def print_metrics(self):
        """Print performance metrics"""
        metrics = self.get_metrics()

        print("Voice Command Performance Metrics:")
        print(f"  Average Response Time: {metrics['avg_response_time']:.3f}s")
        print(f"  Min Response Time: {metrics['min_response_time']:.3f}s")
        print(f"  Max Response Time: {metrics['max_response_time']:.3f}s")
        print(f"  Std Response Time: {metrics['std_response_time']:.3f}s")
        print(f"  Success Rate: {metrics['success_rate']:.2%}")
        print(f"  Total Commands: {metrics['total_commands']}")
```

## Troubleshooting Voice Systems

### Common Voice System Issues and Solutions

```python
class VoiceSystemDiagnostics:
    def __init__(self, node):
        self.node = node
        self.diagnostics = {
            'audio_input': {'status': 'unknown', 'issues': []},
            'speech_recognition': {'status': 'unknown', 'issues': []},
            'nlp_processing': {'status': 'unknown', 'issues': []},
            'command_execution': {'status': 'unknown', 'issues': []}
        }

    def run_diagnostics(self):
        """Run comprehensive diagnostics on voice system"""
        self.check_audio_input()
        self.check_speech_recognition()
        self.check_nlp_processing()
        self.check_command_execution()

        return self.diagnostics

    def check_audio_input(self):
        """Check audio input system"""
        try:
            # Check if audio device is accessible
            import pyaudio
            audio = pyaudio.PyAudio()

            # List audio devices
            device_count = audio.get_device_count()
            if device_count == 0:
                self.diagnostics['audio_input']['status'] = 'error'
                self.diagnostics['audio_input']['issues'].append('No audio devices found')
            else:
                self.diagnostics['audio_input']['status'] = 'ok'
                self.diagnostics['audio_input']['issues'].append(f'Found {device_count} audio devices')

            audio.terminate()
        except Exception as e:
            self.diagnostics['audio_input']['status'] = 'error'
            self.diagnostics['audio_input']['issues'].append(f'Audio system error: {str(e)}')

    def check_speech_recognition(self):
        """Check speech recognition system"""
        try:
            # Check if model is loaded
            if hasattr(self.node, 'model') and self.node.model:
                self.diagnostics['speech_recognition']['status'] = 'ok'
                self.diagnostics['speech_recognition']['issues'].append('Model loaded successfully')
            else:
                self.diagnostics['speech_recognition']['status'] = 'error'
                self.diagnostics['speech_recognition']['issues'].append('Speech recognition model not loaded')
        except Exception as e:
            self.diagnostics['speech_recognition']['status'] = 'error'
            self.diagnostics['speech_recognition']['issues'].append(f'Speech recognition error: {str(e)}')

    def check_nlp_processing(self):
        """Check NLP processing system"""
        try:
            # Test basic NLP functionality
            test_text = "move forward"
            intent, entities = self.node.intent_recognizer.recognize_intent(test_text)

            if intent:
                self.diagnostics['nlp_processing']['status'] = 'ok'
                self.diagnostics['nlp_processing']['issues'].append('NLP processing working')
            else:
                self.diagnostics['nlp_processing']['status'] = 'warning'
                self.diagnostics['nlp_processing']['issues'].append('NLP processing may have issues')
        except Exception as e:
            self.diagnostics['nlp_processing']['status'] = 'error'
            self.diagnostics['nlp_processing']['issues'].append(f'NLP processing error: {str(e)}')

    def check_command_execution(self):
        """Check command execution system"""
        try:
            # Check if publishers are available
            if hasattr(self.node, 'cmd_vel_pub') and self.node.cmd_vel_pub:
                self.diagnostics['command_execution']['status'] = 'ok'
                self.diagnostics['command_execution']['issues'].append('Command execution system ready')
            else:
                self.diagnostics['command_execution']['status'] = 'error'
                self.diagnostics['command_execution']['issues'].append('Command execution system not ready')
        except Exception as e:
            self.diagnostics['command_execution']['status'] = 'error'
            self.diagnostics['command_execution']['issues'].append(f'Command execution error: {str(e)}')

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

## Best Practices for Voice-to-Action Systems

### 1. Safety and Security

- Always implement emergency stop capabilities
- Validate commands against safety constraints
- Implement authentication for sensitive commands
- Use encrypted communication for voice data

### 2. User Experience

- Provide clear audio/visual feedback for voice commands
- Implement wake word detection to avoid accidental triggers
- Design intuitive command grammars
- Handle ambiguous commands gracefully

### 3. Performance Optimization

- Use appropriate models for your hardware constraints
- Implement streaming processing for real-time responses
- Optimize for low-latency command execution
- Consider offline processing for critical commands

## Exercises

1. **Voice Recognition Setup**: Implement a voice recognition system using Vosk or another speech recognition library that can reliably recognize basic robot commands.

2. **Command Grammar Exercise**: Design and implement a grammar for voice commands that covers navigation, manipulation, and interaction tasks for a humanoid robot.

3. **Integration Exercise**: Create a complete voice-to-action pipeline that integrates speech recognition, NLP processing, and robot command execution with proper error handling and safety checks.

## Summary

This chapter covered voice-to-action systems for humanoid robotics, including speech recognition technologies, natural language processing, command validation, and ROS 2 integration. We explored advanced techniques like wake word detection and context management, and discussed performance evaluation and troubleshooting approaches. Voice-to-action systems provide an intuitive interface for human-robot interaction, making humanoid robots more accessible and user-friendly.

## Further Reading

- [Speech Recognition with Vosk](https://alphacephei.com/vosk/)
- [ROS 2 Audio Processing](https://index.ros.org/p/audio_common/)
- [Natural Language Processing for Robotics](https://arxiv.org/abs/2103.12531)
- [Human-Robot Interaction Guidelines](https://ieeexplore.ieee.org/document/9109381)

---

**Next Chapter**: [LLM-Based Cognitive Planning](./llm-planning.md)