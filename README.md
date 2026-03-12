# Flask Migrations

This repo demonstrates how to use Flask-Migrate to manage database schema changes in a Flask application. It includes exercises to practice creating and applying migrations, as well as evolving the database schema over time. The README provides step-by-step instructions for setting up the project, running migrations, and testing the API endpoints.

## To initialize the database run from repo root:
```bash
flask --app src.migration_demo.manage:app db init 
```
Check that the `migrations/` folder is created.
## To generate a migration after making changes to the models:
```bash
flask --app src.migration_demo.manage:app db migrate -m "Initial migration"
```
Check that a new migration file is created in the `migrations/versions/` folder with the expected schema changes.
## To apply the migration to the database:
```bash
flask --app src.migration_demo.manage:app db upgrade
```
Check that sqlite3 database file is created in the data directory and that the schema matches the models defined in `app.py`. You can also test the API endpoints to confirm that the application is working as expected.

Now run the demo application:
```bash
flask --app src.migration_demo.manage:app run
```
You can test the API endpoints using curl or Postman to confirm that the application is working as

Load initial data by running 
```bash
scripts/create_initial_data.sh
```
or 
```cmd
scripts\create_initial_data.bat
```

Change model definition for student by adding a cohort field:
```python
class Student(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(128), nullable=False)
    # add this new field to the model
    cohort = db.Column(db.String(128), nullable=True)  # New field for cohort
```

Then generate a new migration to reflect this change:
```bash
flask --app src.migration_demo.manage:app db migrate -m "Add cohort field to Student model"
```
Apply the migration:
```bash
flask --app src.migration_demo.manage:app db upgrade
```
Now the database schema should be updated to include the new `cohort` field in the `students` table. 
We have to  update the API endpoints to allow setting and retrieving the cohort information for students.

We will update the POST endpoint to create a student with a cohort:
```python
@app.route('/students', methods=['POST'])
def create_student():
    data = request.get_json()
    name = data.get('name')
    cohort = data.get('cohort')  # Get cohort from request data
    if not name:
        return jsonify({'error': 'Name is required'}), 400
    student = Student(name=name, cohort=cohort)  # Set cohort when creating student
    db.session.add(student)
    db.session.commit()
    return jsonify({'id': student.id, 'name': student.name, 'cohort': student.cohort}), 201  # Include cohort in response
```
And we will update the GET endpoint to include cohort information in the response:
```python   
@app.route('/students', methods=['GET'])
def get_students():
    students = Student.query.all()
    return jsonify([{'id': student.id, 'name': student.name, 'cohort': student.cohort} for student in students])  # Include cohort in response
```     
We can now test the updated API endpoints to confirm that we can create students with cohort information and retrieve it correctly.