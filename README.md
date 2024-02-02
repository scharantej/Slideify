## Flask Application Design

### Problem:
Build a simple Flask web application that allows users to create and manage tasks.

### Design:

#### HTML Files:

**index.html:**
- Serves as the main page of the application.
- Contains a form for users to input and submit new tasks.
- HTML:
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Task Manager</title>
  </head>
  <body>
    <form action="/create" method="POST">
      <input type="text" name="task">
      <input type="submit" value="Add">
    </form>

    <ul id="tasks">
    </ul>

    <script>
      const taskList = document.getElementById('tasks');

      // Function to create and add a new task to the list
      const addTask = (task) => {
        const newListItem = document.createElement('li');
        newListItem.textContent = task;
        taskList.appendChild(newListItem);
      }

      // Event listener for the form submit event
      document.querySelector('form').addEventListener('submit', (e) => {
        e.preventDefault();
        const task = e.target.querySelector('input[name="task"]').value;
        addTask(task);
        // Send the task to the server using fetch
        fetch('/create', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({ task })
        });
      });
    </script>
  </body>
</html>
```

#### Routes:

**app.py:**

- Serves as the entry point for the Flask application.
- Defines routes for handling requests.

```python
from flask import Flask, request, render_template, redirect, url_for
import sqlite3

app = Flask(__name__)

# Database connection
conn = sqlite3.connect('tasks.db')
c = conn.cursor()

# Create table if not exists
c.execute('''CREATE TABLE IF NOT EXISTS tasks (
              id INTEGER PRIMARY KEY,
              task TEXT NOT NULL
            )''')

# Route for creating a new task
@app.route('/create', methods=['POST'])
def create_task():
    task = request.json.get('task')
    c.execute("INSERT INTO tasks (task) VALUES (?)", (task,))
    conn.commit()
    return redirect(url_for('index'))

# Route for displaying the homepage
@app.route('/')
def index():
    tasks = c.execute("SELECT * FROM tasks").fetchall()
    return render_template('index.html', tasks=tasks)

if __name__ == '__main__':
    app.run(debug=True)
```

### Explanation:

- **HTML:** The index.html file serves as the application's main interface. It provides a form for users to input new tasks and a list to display the tasks.
- **Routes:** The app.py file defines two routes:

 - The **'/create'** route handles requests to create new tasks. It receives the task from the request, adds it to the database, and redirects the user to the homepage.
 - The **'/'** route serves the homepage, displaying the list of tasks from the database.