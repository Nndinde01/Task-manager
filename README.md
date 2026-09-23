# Task Manager

A command-line task manager written in Python. It lets you create tasks, track their status and priority, manage due dates and tags, and view task statistics. Tasks are stored in a local JSON file so they remain available between runs.

## Requirements

- Python 3.8 or later
- No third-party packages

## Project structure

```text
Task-manager/
├── app.py       # TaskManager service and task operations
├── cli.py       # Command-line interface
├── models.py    # Task model, statuses, and priorities
├── storage.py   # JSON persistence
└── README.md
```

Run the CLI from the directory that contains these files, as a Python package:

```bash
python -m Task-manager
```

> Python package names cannot contain hyphens. If this folder is literally named `Task-manager`, the command above will not work. From its parent directory, rename it to `task_manager` and run `python -m task_manager.cli`, or use the equivalent package name after renaming. The commands below assume you are in the package's parent directory and the package is named `task_manager`.

## Usage

```bash
python -m task_manager.cli --help
```

Each command has its own help page. For example:

```bash
python -m task_manager.cli create --help
```

The application creates `tasks.json` in the current working directory the first time it saves a task. You can choose another file by constructing `TaskManager(storage_path="path/to/file.json")` in Python; the CLI uses the default `tasks.json` path.

### Create a task

```bash
python -m task_manager.cli create "Prepare release" \
  --description "Review the final changes" \
  --priority 3 \
  --due 2026-10-15 \
  --tags release,review
```

The title is required. Description defaults to empty, priority defaults to `2` (medium), and due date and tags are optional. Due dates use `YYYY-MM-DD`. Priority values are:

| Value | Priority |
| --- | --- |
| 1 | Low |
| 2 | Medium |
| 3 | High |
| 4 | Urgent |

### List tasks

```bash
python -m task_manager.cli list
python -m task_manager.cli list --status in_progress
python -m task_manager.cli list --priority 4
python -m task_manager.cli list --overdue
```

Status values are `todo`, `in_progress`, `review`, and `done`. A list command accepts one filter at a time. If more than one is supplied, overdue takes precedence over status, and status takes precedence over priority. Priority filtering matches a value; it does not sort the results.

### View and update a task

```bash
python -m task_manager.cli show TASK_ID
python -m task_manager.cli status TASK_ID in_progress
python -m task_manager.cli status TASK_ID done
python -m task_manager.cli priority TASK_ID 4
python -m task_manager.cli due TASK_ID 2026-10-20
```

The list and show output displays the first eight characters of a task ID for readability. Commands that look up a task require its full UUID, which is printed when the task is created. Keep that full ID if you need to update or delete the task.

Marking a task `done` sets its completion timestamp. Moving a task back to another status does not clear that timestamp.

### Manage tags

```bash
python -m task_manager.cli tag TASK_ID planning
python -m task_manager.cli untag TASK_ID planning
```

Adding an existing tag has no effect. Removing a tag that is not on the task reports a failure.

### Delete a task

```bash
python -m task_manager.cli delete TASK_ID
```

Deletion removes the task from the local JSON store.

### View statistics

```bash
python -m task_manager.cli stats
```

Statistics include the total number of tasks, counts by status and priority, the number currently overdue, and tasks completed in the last seven days. Priority counts are keyed by numeric values (`1` through `4`); status counts use the status strings.

## Data and behavior

Each task has a UUID, title, description, priority, status, creation and update timestamps, an optional due date, an optional completion timestamp, and a list of tags. New tasks start in `todo` status with medium priority unless another priority is selected.

The storage layer keeps tasks in memory while the program runs and writes them to a JSON file after changes. Dates and times are serialized as ISO 8601 strings. The default file is `tasks.json` in the current working directory. Keep a backup of this file if the task data matters to you.

A task is overdue when it has a due date earlier than the current time and is not marked done. Due dates entered by the CLI are parsed at midnight on the selected date.

## Current limitations

- Task IDs must be entered in full for commands that retrieve a task, even though display output shortens them.
- Filtering does not combine criteria, and the general list is not sorted by priority.
- Storage errors are printed to the console. The application does not provide transactional rollback if a write fails.
- This is a local file-based CLI; it has no user accounts, shared access, or remote synchronization.

## Python API

The main service can also be used from another Python module:

```python
from task_manager.app import TaskManager

manager = TaskManager("tasks.json")
task_id = manager.create_task(
    title="Prepare release",
    description="Review the final changes",
    priority_value=3,
    due_date_str="2026-10-15",
    tags=["release", "review"],
)
```


`TaskManager` provides methods for creating, listing, updating, retrieving, deleting, and reporting on tasks. See `app.py`, `models.py`, and `storage.py` for their implementation.

Understanding a Specific Feature
Exercise Part 1: Task Creation & Updates
Primary Components
 ⁠cli.py⁠: Handles user input via ⁠argparse⁠ and formats output.  
 ⁠task_manager.py⁠: Acts as the controller/service layer, validating parameters and coordinating business logic.  
 ⁠storage.py⁠: Encapsulates persistence using JSON loading/saving via custom encoder/decoder classes (⁠TaskEncoder⁠, ⁠TaskDecoder⁠).  
 ⁠models.py⁠: Defines core domain entities (⁠Task⁠, ⁠TaskPriority⁠, ⁠TaskStatus⁠) and state modification logic.

 Execution Flow (Task Creation & Status Updates)
1. Creation: ⁠cli.py⁠ receives args \rightarrow calls ⁠TaskManager.create_task()⁠ \rightarrow instantiates ⁠Task⁠ with a generated UUID4 \rightarrow ⁠TaskStorage.add_task()⁠ appends task to memory map and triggers ⁠save()⁠ to dump JSON to disk.  
2. Status Update: ⁠cli.py⁠ receives args \rightarrow calls ⁠TaskManager.update_task_status()⁠. If status is set to ⁠DONE⁠, it invokes ⁠Task.mark_as_done()⁠, setting both ⁠status⁠ and ⁠completed_at⁠ before saving. Otherwise, it invokes ⁠Task.update()⁠, modifying attributes and setting ⁠updated_at⁠ my files.

Data Storage & Design Patterns
 Data Storage: In-memory Python dictionary (⁠self.tasks⁠) backed by disk persistence via ⁠tasks.json⁠.  
 Design Patterns:
 Repository / DAO Pattern: ⁠TaskStorage⁠ abstracts low-level file I/O away from business operations.  
 Facade Pattern: ⁠TaskManager⁠ simplifies interactions with models and storage into unified service methods.  
 Custom Serialization: Extends ⁠json.JSONEncoder⁠ and ⁠json.JSONDecoder⁠ to handle custom domain models and ⁠datetime⁠ conversions.

 PART 2

 My Initial Understanding
Task priorities are backed by the ⁠TaskPriority⁠ Enum with four integer values: ⁠LOW = 1⁠, ⁠MEDIUM = 2⁠, ⁠HIGH = 3⁠, ⁠URGENT = 4⁠. When listing tasks by priority, filtering retrieves items where ⁠task.priority⁠ matches the given priority Enum.

key insights the guided questions
Does the storage engine or list filter support sorting tasks by priority (e.g., displaying ⁠URGENT⁠ first), or does it only perform exact-match filtering?
3. How are priority counts calculated in ⁠get_statistics()⁠, and does it use integer values or Enum names?

Key Insights & Misconception Clarification
 Insight: CLI input is strictly validated by ⁠argparse⁠ choices ⁠[1, 2, 3, 4]⁠ before hitting ⁠TaskManager⁠, where ⁠TaskPriority(value)⁠ converts integers to Enums.  
 Misconception Clarified: The system filters by priority via ⁠get_tasks_by_priority()⁠, but does not sort task lists dynamically by priority rank when calling general listing commands.  
 Statistics Difference: In statistics, ⁠by_status⁠ keys use string values (⁠status.value⁠), whereas ⁠by_priority⁠ keys use Enum names (⁠priority.name⁠).

 PART 3
Input Parsing (⁠cli.py⁠): Captures the ⁠task_id⁠ and new status string from command-line arguments and passes them to ⁠task_manager.update_task_status()⁠.  
2. Branching Logic (⁠task_manager.py⁠): Converts the status string into a ⁠TaskStatus⁠ Enum:  
 If ⁠TaskStatus.DONE⁠: Retrieves the ⁠Task⁠ object from storage and calls its ⁠mark_as_done()⁠ method.  
 If any other status (⁠TODO⁠, ⁠IN_PROGRESS⁠, ⁠REVIEW⁠): Calls ⁠TaskStorage.update_task()⁠, which delegates to ⁠Task.update()⁠.  
3. State Mutation (⁠models.py⁠):
 ⁠mark_as_done()⁠ transitions ⁠status⁠ to ⁠TaskStatus.DONE⁠, updates ⁠updated_at⁠, and sets ⁠completed_at⁠ to ⁠datetime.now()⁠.  
 ⁠update()⁠ modifies attributes and refreshes ⁠updated_at⁠.  
4. Persistence Refresh (⁠storage.py⁠): ⁠TaskStorage.save()⁠ is called to re-serialize the mutated task list and write the changes to ⁠tasks.json⁠.
 State Changes During Task Completion
 ⁠task.status⁠: ⁠TaskStatus.TODO⁠ / ⁠IN_PROGRESS⁠ / ⁠REVIEW⁠ \rightarrow ⁠TaskStatus.DONE⁠.  
 ⁠task.completed_at⁠: ⁠None⁠ \rightarrow ⁠datetime.now()⁠.  
 ⁠task.updated_at⁠: Old timestamp \rightarrow ⁠datetime.now()⁠.

 Potential Failure Points
 File I/O Errors: ⁠TaskStorage.save()⁠ catches exceptions generically (⁠except Exception as e⁠) and prints an error message, but does not re-raise or rollback state, causing in-memory state to desynchronize from the JSON file.  
 UUID Truncation Match: CLI users copy 8-character partial UUIDs from ⁠format_task()⁠, but ⁠TaskStorage.get_task()⁠ expects full UUID strings for exact dictionary key lookups, causing lookup failures unless full IDs are passed.

 PART 4
 How Core Features Work
 Task Creation: Generates UUID4, initializes timestamps, and commits to file storage.  
 Task Prioritization: Uses strongly typed Enums (⁠TaskPriority⁠) for CLI parameter choices and filter predicates.  
 Task Completion: Invokes domain method ⁠mark_as_done()⁠ to transition status, update timestamps, and capture ⁠completed_at⁠.

 Exercise: Algorithm Deconstruction Challenge

Algorithm 1: Task Scoring & Priority Ranking (⁠calculate_task_score⁠)
Calculates a numeric importance score to rank and prioritize tasks dynamically.
Algorithm 2: Natural Text Task Parser (⁠parse_task_from_text⁠)
Parses unstructured human text strings containing inline tokens into structured ⁠Task⁠ entities.

 Insights and Learning Points
 Lack of Soft-Deletions (Tombstones): Hard-deleting a task locally removes its key from ⁠local_tasks⁠. During the next merge, the engine interprets this as a "Remote Only" entry and restores the deleted task from the remote store. A production sync engine requires soft-deletion flags (e.g., ⁠is_deleted = True⁠) to track deletions across clients.  
 Coarse Entity Overwrites: The algorithm checks timestamps at the entity level. If User A edits a task's title locally and User B edits its due date remotely a minute later, User B's object overwrites User A's title change entirely rather than merging the individual updated fields.  
 Decoupled Side Effects: Returning ⁠(merged_tasks, sync_actions)⁠ cleanly isolates data manipulation from audit logging, keeping the code modular and easy to unit test.

 Exercise: Code Documentation

 Original code
 @app.route('/api/users/register', methods=['POST'])
def register_user():
    data = request.get_json()

    username = data.get('username')
    email = data.get('email')
    password = data.get('password')

    if not username or not email or not password:
        return jsonify({
            'error': 'Username, email and password are required'
        }), 400

    # Create the user
    user = {
        'username': username,
        'email': email,
        'password': password
    }

    return jsonify({
        'message': 'User registered successfully',
        'user': user
    }), 201

    Documentation using prompt 1
    Comprehensive Function Documentation

Function: register_user()

Description

The register_user() function registers a new user through the API. It receives the user’s username, email address, and password as JSON data.

The function checks that all required information has been provided. If information is missing, it returns an error response. If all information is provided, it creates a user object and returns a successful registration response.

Endpoint

POST /api/users/register

Parameters

The function receives JSON data in the request body.

Parameter	Type	Description
username	String	The username of the new user.
email	String	The email address of the new user.
password	String	The password of the new user.

Return Values

Successful response

Status code: 201 Created

Returns a JSON response containing:

* A success message
* The newly created user’s information

Error response

Status code: 400 Bad Request

Returned when the username, email, or password is missing.

Possible Errors

The function may return an error when:

* No JSON data is supplied.
* The username is missing.
* The email is missing.
* The password is missing.

Example Request

{
    "username": "pretty",
    "email": "pretty@example.com",
    "password": "mypassword"
}

Example Successful Response

{
    "message": "User registered successfully",
    "user": {
        "username": "pretty",
        "email": "pretty@example.com",
        "password": "mypassword"
    }
}

Example Error Response

{
    "error": "Username, email and password are required"
}

The purpose of register_user() is to allow a new user to register with the application.

The function receives registration information, checks whether the required information exists, creates a user object, and sends a response to the client.

2. Step-by-Step Logic

Step 1: Define the endpoint

@app.route('/api/users/register', methods=['POST'])

This creates the registration API endpoint and allows clients to send POST requests.

Step 2: Define the function

def register_user():

This function contains the registration logic.

Step 3: Read the request data

data = request.get_json()

The function reads the JSON information sent by the client.

Step 4: Get the user’s information

username = data.get('username')
email = data.get('email')
password = data.get('password')

The function retrieves the username, email and password from the JSON data.

Step 5: Validate required fields

if not username or not email or not password:

The function checks whether any required information is missing.

If something is missing, it returns:

return jsonify({
    'error': 'Username, email and password are required'
}), 400

The 400 status code means the request was invalid.

Step 6: Create the user

user = {
    'username': username,
    'email': email,
    'password': password
}

A user object is created using the supplied information.

Step 7: Return a successful response

return jsonify({
    'message': 'User registered successfully',
    'user': user
}), 201

The function returns a success message and uses status code 201, which indicates that a resource was created.

3. Assumptions and Edge Cases

The implementation assumes:

* The client sends JSON data.
* The JSON contains the expected field names.
* The username, email and password are provided.
* The information is valid.

Possible edge cases include:

* Empty JSON requests.
* Invalid email addresses.
* Weak passwords.
* Duplicate usernames.
* Duplicate email addresses.
* Passwords being stored as plain text.

4. Suggested Inline Comments

# Read JSON data sent by the client
data = request.get_json()
# Get registration information
username = data.get('username')
email = data.get('email')
password = data.get('password')
# Check that all required fields were provided
if not username or not email or not password:
    return jsonify({
        'error': 'Username, email and password are required'
    }), 400
# Create the new user
user = {
    'username': username,
    'email': email,
    'password': password
}

5. Potential Improvements

The original functionality can be maintained while improving security and reliability by:

1. Hashing the password instead of storing it as plain text.
2. Validating the email address.
3. Checking whether the username already exists.
4. Checking whether the email is already registered.
5. Handling requests that contain invalid JSON.
6. Adding password-strength requirements.
7. Storing users in a database instead of only creating an in-memory object.

User Registration API Documentation

Overview

The User Registration API allows a new user to create an account by providing a username, email address, and password.

Endpoint

POST /api/users/register

Function

register_user()

Request

The endpoint expects JSON data containing three fields:

{
    "username": "pretty",
    "email": "pretty@example.com",
    "password": "mypassword"
}

Required Parameters

Parameter	Type	Required	Description
username	String	Yes	Username for the new account.
email	String	Yes	Email address for the account.
password	String	Yes	Password for the account.

How the Function Works

1. The client sends a POST request to /api/users/register.
2. The function reads the JSON request.
3. It extracts the username, email and password.
4. It checks whether all three fields have been provided.
5. If information is missing, it returns a 400 Bad Request.
6. If all required information is present, a user object is created.
7. The API returns a 201 Created response.

Successful Response

Status: 201 Created

{
    "message": "User registered successfully",
    "user": {
        "username": "pretty",
        "email": "pretty@example.com",
        "password": "mypassword"
    }
}

Error Response

Status: 400 Bad Request

{
    "error": "Username, email and password are required"
}

Important Edge Cases

The function should consider:

* Missing username.
* Missing email.
* Missing password.
* Empty request data.
* Invalid email addresses.
* Duplicate usernames.
* Duplicate email addresses.
* Weak passwords.
* Invalid JSON.

Security Considerations

The example code should be improved before being used in a real application. In particular, passwords should be securely hashed and should not be returned in API responses.

Suggested Documentation Comment

def register_user():
    """
    Register a new user.
    Reads username, email, and password from a JSON request,
    validates that all required fields are present, and creates
    a new user object.
    Returns:
        Response: JSON success response with status 201 when
        registration succeeds, or an error response with status
        400 when required information is missing.
    """

Overall Purpose

The main purpose of this function is to handle user registration by receiving user details, validating required fields, creating a user object, and returning an appropriate HTTP response.


Exercise: API Documentation
original API ENDPOINT 
@app.route('/api/users/register', methods=['POST'])
def register_user():
    data = request.get_json()

    username = data.get('username')
    email = data.get('email')
    password = data.get('password')

    if not username or not email or not password:
        return jsonify({
            'error': 'Username, email and password are required'
        }), 400

    user = {
        'username': username,
        'email': email,
        'password': password
    }

    return jsonify({
        'message': 'User registered successfully',
        'user': user
    }), 201

    PROMPT 1 DOCUMENTATION
    openapi: 3.0.0

info:
  title: User Registration API
  description: API for registering new users.
  version: 1.0.0

servers:
  - url: http://localhost:5000

paths:
  /api/users/register:
    post:
      summary: Register a new user
      description: Creates a new user account using a username, email and password.

      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserRegistration'

            example:
              username: pretty
              email: pretty@example.com
              password: mypassword

      responses:

        '201':
          description: User registered successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SuccessResponse'

              example:
                message: User registered successfully
                user:
                  username: pretty
                  email: pretty@example.com
                  password: mypassword

        '400':
          description: Required information is missing
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

              example:
                error: Username, email and password are required

components:

  schemas:

    UserRegistration:
      type: object
      required:
        - username
        - email
        - password

      properties:
        username:
          type: string
          description: Username of the new user
          example: pretty

        email:
          type: string
          format: email
          description: Email address of the new user
          example: pretty@example.com

        password:
          type: string
          format: password
          description: Password for the new account
          example: mypassword

    SuccessResponse:
      type: object
      properties:
        message:
          type: string
          example: User registered successfully

        user:
          type: object
          properties:
            username:
              type: string
              example: pretty

            email:
              type: string
              example: pretty@example.com

            password:
              type: string
              example: mypassword

    ErrorResponse:
      type: object
      properties:
        error:
          type: string
          example: Username, email and password are required 

    PROMPT 3
    The User Registration API allows a new user to create an account by providing a username, email address, and password.

    Exercise: README and User Guide Documentation

    /task_manager
task_manager.py     # Main application executable entry point
tasks.txt           # Text file database storing task records
user.txt            # Text file database storing user credentials
task_overview.txt   # Generated report on overall task statistics
user_overview.txt   # Generated report on per-user task statistics

# Task Manager System

A lightweight, CLI-based task management application written in Python to help teams assign, track, and report on operational tasks.

## Description
The Task Manager System provides a simple command-line interface for managing individual and team tasks. It reads and persists data using text files (`tasks.txt` and `user.txt`), allowing users to register, create tasks, edit incomplete tasks, and generate executive summaries on project statistics.

## Key Features
- **User Authentication:** Secure login mechanism verifying credentials against stored user data.
- **Task Management:** Add new tasks, assign them to specific team members, set due dates, and mark tasks as complete.
- **Task Editing:** Modify assigned users or due dates for tasks that have