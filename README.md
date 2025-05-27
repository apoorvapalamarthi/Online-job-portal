import webbrowser
from flask import Flask, request, render_template_string, redirect, url_for

app = Flask(__name__)

# In-memory data stores
jobs = []
applications = []

# HTML Templates embedded in the code
home_page = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Job Portal</title>
    <style>
        body {
            background-color: #007BFF;
            color: white;
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        h1, h2 {
            text-align: center;
        }
        a {
            color: white;
            text-decoration: none;
            background-color: #0056b3;
            padding: 10px 20px;
            border-radius: 5px;
        }
        a:hover {
            background-color: #004085;
        }
        ul {
            list-style-type: none;
            padding: 0;
        }
        li {
            background-color: #0056b3;
            margin: 10px 0;
            padding: 15px;
            border-radius: 8px;
        }
        li h3 {
            margin: 0;
            font-size: 1.5em;
        }
        p {
            font-size: 1.1em;
        }
    </style>
</head>
<body>
    <h1>Welcome to the Job Portal</h1>
    <a href="{{ url_for('post_job') }}">Post a New Job</a>
    <h2>Available Jobs</h2>
    <ul>
        {% for job in jobs %}
            <li>
                <h3>{{ job.title }}</h3>
                <p>{{ job.description }}</p>
                <p>Location: {{ job.location }}</p>
                <a href="{{ url_for('apply', job_id=loop.index0) }}">Apply</a>
                <a href="{{ url_for('view_applications', job_id=loop.index0) }}">View Applications</a>
            </li>
        {% else %}
            <li>No jobs posted yet.</li>
        {% endfor %}
    </ul>
</body>
</html>
"""

post_job_page = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Post a Job</title>
    <style>
        body {
            background-color: #007BFF;
            color: white;
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        form {
            max-width: 600px;
            margin: 0 auto;
            background-color: #0056b3;
            padding: 20px;
            border-radius: 8px;
        }
        label {
            font-size: 1.1em;
            margin-bottom: 10px;
            display: block;
        }
        input, textarea {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border-radius: 5px;
            font-size: 1em;
        }
        button {
            background-color: #004085;
            color: white;
            padding: 10px 20px;
            border-radius: 5px;
            border: none;
            font-size: 1.1em;
        }
        button:hover {
            background-color: #003366;
        }
    </style>
</head>
<body>
    <h1>Post a New Job</h1>
    <form method="POST">
        <label for="title">Job Title:</label>
        <input type="text" name="title" id="title" required>
        <br>
        <label for="description">Job Description:</label>
        <textarea name="description" id="description" required></textarea>
        <br>
        <label for="location">Location:</label>
        <input type="text" name="location" id="location" required>
        <br>
        <button type="submit">Post Job</button>
    </form>
</body>
</html>
"""

apply_page = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Apply for Job</title>
    <style>
        body {
            background-color: #007BFF;
            color: white;
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        form {
            max-width: 600px;
            margin: 0 auto;
            background-color: #0056b3;
            padding: 20px;
            border-radius: 8px;
        }
        label {
            font-size: 1.1em;
            margin-bottom: 10px;
            display: block;
        }
        input {
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            border-radius: 5px;
            font-size: 1em;
        }
        button {
            background-color: #004085;
            color: white;
            padding: 10px 20px;
            border-radius: 5px;
            border: none;
            font-size: 1.1em;
        }
        button:hover {
            background-color: #003366;
        }
    </style>
</head>
<body>
    <h1>Apply for {{ job.title }}</h1>
    <form method="POST">
        <label for="name">Name:</label>
        <input type="text" name="name" id="name" required>
        <br>
        <label for="email">Email:</label>
        <input type="email" name="email" id="email" required>
        <br>
        <button type="submit">Submit Application</button>
    </form>
</body>
</html>
"""

view_applications_page = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Applications for {{ job.title }}</title>
    <style>
        body {
            background-color: #007BFF;
            color: white;
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        h1 {
            text-align: center;
        }
        ul {
            list-style-type: none;
            padding: 0;
        }
        li {
            background-color: #0056b3;
            margin: 10px 0;
            padding: 15px;
            border-radius: 8px;
        }
        li h3 {
            margin: 0;
            font-size: 1.5em;
        }
        p {
            font-size: 1.1em;
        }
    </style>
</head>
<body>
    <h1>Applications for {{ job.title }}</h1>
    <ul>
        {% for application in applications %}
            <li>
                <h3>{{ application.name }}</h3>
                <p>Email: {{ application.email }}</p>
            </li>
        {% else %}
            <li>No applications yet.</li>
        {% endfor %}
    </ul>
</body>
</html>
"""

# Routes
@app.route('/')
def home():
    return render_template_string(home_page, jobs=jobs)

@app.route('/post_job', methods=['GET', 'POST'])
def post_job():
    if request.method == 'POST':
        job = {
            'title': request.form['title'],
            'description': request.form['description'],
            'location': request.form['location'],
            'applications': []  # Initialize empty list for applications
        }
        jobs.append(job)
        return redirect(url_for('home'))
    return render_template_string(post_job_page)

@app.route('/apply/<int:job_id>', methods=['GET', 'POST'])
def apply(job_id):
    job = jobs[job_id]
    if request.method == 'POST':
        application = {
            'name': request.form['name'],
            'email': request.form['email']
        }
        job['applications'].append(application)  # Store application under the specific job
        return redirect(url_for('home'))
    return render_template_string(apply_page, job=job)

@app.route('/applications/<int:job_id>')
def view_applications(job_id):
    job = jobs[job_id]
    return render_template_string(view_applications_page, job=job, applications=job['applications'])

if __name__ == '__main__':
    # Get the address where the app will run
    url = "http://127.0.0.1:5000/"
    
    # Open the default web browser automatically
    webbrowser.open(url, new=2)
    
    # Start the Flask application
    app.run(debug=True)
