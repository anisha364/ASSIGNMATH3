from flask import Flask, render_template, request, redirect
import sqlite3
from datetime import datetime

app = Flask(__name__)

def get_db():
    return sqlite3.connect("database.db")

@app.route("/", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        role = request.form["role"]
        return redirect("/" + role)
    return render_template("login.html")

@app.route("/teacher", methods=["GET", "POST"])
def teacher():
    db = get_db()
    cursor = db.cursor()

    if request.method == "POST":
        title = request.form["title"]
        deadline = request.form["deadline"]
        cursor.execute(
            "INSERT INTO assignments (title, deadline) VALUES (?, ?)",
            (title, deadline)
        )
        db.commit()

    assignments = cursor.execute("SELECT * FROM assignments").fetchall()
    return render_template("teacher.html", assignments=assignments)

@app.route("/student", methods=["GET", "POST"])
def student():
    db = get_db()
    cursor = db.cursor()

    if request.method == "POST":
        assignment_id = request.form["assignment_id"]
        submit_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        cursor.execute(
            "INSERT INTO submissions (assignment_id, submit_time) VALUES (?, ?)",
            (assignment_id, submit_time)
        )
        db.commit()

    assignments = cursor.execute("SELECT * FROM assignments").fetchall()
    return render_template("student.html", assignments=assignments)

if __name__ == "__main__":
    db = get_db()
    cursor = db.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS assignments (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT,
            deadline TEXT
        )
    """)
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS submissions (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            assignment_id INTEGER,
            submit_time TEXT
        )
    """)
    db.commit()

    app.run(host="0.0.0.0", port=3000)

