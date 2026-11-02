# CampusCare — Smart Complaint Management System

A moderate-level MCA Advanced Internet Programming mini project built with Flask, MySQL, Bootstrap, SQLAlchemy, and an ML category classifier.

## Features
- Student registration/login with Werkzeug password hashing and role-based access.
- Complaint submission, unique reference, category prediction, optional image, search, tracking timeline, withdrawal, reopening window, and feedback.
- Admin dashboard with database-backed metrics and Chart.js charts.
- Admin case management with status transition validation, public comments, restricted internal notes, assignment, account activation/deactivation, feedback review, and CSV export.
- CSRF protection, ORM queries, upload extension and size checks, and ownership checks.
- TF-IDF + Multinomial Naive Bayes training pipeline and a keyword fallback if model artifacts are absent.
- Pytest tests for core behavior.

## Requirements
- Python 3.10+ (3.11 recommended)
- MySQL Server 8+
- MySQL Workbench (optional GUI)
- VS Code

## 1. Create the MySQL database
Open MySQL Workbench, connect to your local server, and run:

```sql
CREATE DATABASE smart_complaints CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'complaint_user'@'localhost' IDENTIFIED BY 'Choose-A-Strong-Password';
GRANT ALL PRIVILEGES ON smart_complaints.* TO 'complaint_user'@'localhost';
FLUSH PRIVILEGES;
```

If your MySQL account uses a different host, adjust the user host and `DATABASE_URL` accordingly. Keep real credentials out of source control.

## 2. Open the folder in VS Code
Extract the ZIP, then in VS Code choose **File → Open Folder** and select `smart_complaint_system`.

Open Terminal → New Terminal. The following commands are for Windows PowerShell:

```powershell
py --version
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
Copy-Item .env.example .env
```

If PowerShell blocks activation, run this for the current terminal only:
`Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`
Or use Command Prompt activation: `.venv\\Scripts\\activate.bat`.

## 3. Configure environment
Open `.env` and set `DATABASE_URL` to your MySQL credentials. Example:

```text
DATABASE_URL=mysql+pymysql://complaint_user:YourPassword@127.0.0.1:3306/smart_complaints
SECRET_KEY=replace-with-a-long-random-secret
FLASK_DEBUG=true
COOKIE_SECURE=false
REOPEN_WINDOW_DAYS=7
```

Use a long random `SECRET_KEY`. If the database password includes URL-special characters, URL-encode them. `COOKIE_SECURE=false` is suitable only for local HTTP development; use HTTPS and `true` in deployment.

## 4. Create database tables and admin
In the VS Code terminal, from the project root:

```powershell
flask --app run.py init-db
flask --app run.py create-admin
```

Enter a valid admin email, name, and password (minimum 10 characters) when prompted. Student registration cannot create an admin role.

## 5. Train the ML model (optional but recommended)
The application uses a rule-based fallback until model files exist. To train the actual TF-IDF + Naive Bayes model:

```powershell
python ml\train_model.py
```

The script saves artifacts to `ml/artifacts/`. This demonstration dataset is small and synthetic; its evaluation scores do **not** indicate real-world accuracy. For better results, add accurately labeled, diverse real complaint examples to `ml/dataset/complaints.csv` and retrain.

## 6. Run the application
```powershell
python run.py
```
Open `http://127.0.0.1:5000` in your browser. Register a student account, submit a test complaint, then log in with the admin account to update status and comments.

## 7. Run tests
The included tests use a temporary SQLite in-memory database only for isolated automated tests. The actual application configuration and normal execution use MySQL as required.

```powershell
pytest -q
```

The test fixture overrides the database URI and does not modify your development MySQL database.

## Manual demo checklist
1. Register a student and log in.
2. Submit a complaint with all required fields.
3. Confirm a reference number appears and complaint is in the student dashboard.
4. Search by title/reference and open the tracking timeline.
5. Log in as admin and verify complaint appears in admin list and analytics.
6. Change status to Acknowledged, then In Progress, then Resolved; include a public comment.
7. Verify student can see public comments but not internal notes.
8. Submit feedback after resolution.
9. Check feedback in the admin panel.
10. Export CSV and open it in a spreadsheet.
11. Try accessing admin pages as a student and verify access is denied.
12. Test invalid form fields and disallowed image extensions.

## Troubleshooting
- **MySQL connection refused:** ensure MySQL Server service is running, port 3306 is open, and `DATABASE_URL` uses the correct host/port.
- **Access denied:** check database username/password and `GRANT` permissions.
- **Unknown database:** create `smart_complaints` using the SQL above.
- **No ML model found:** normal before training; run `python ml\\train_model.py`.
- **Missing package:** activate `.venv`, then `pip install -r requirements.txt`.
- **Port already in use:** stop the other Flask process or run on another port using `app.run(port=5001)`.
- **Template error:** run from the project root and confirm all files were extracted, then inspect terminal traceback.
- **Image upload rejected:** only PNG/JPG/JPEG/GIF/WEBP, maximum 5 MB.

## Architecture
- `routes/`: public, authentication, student, and administrator endpoints.
- `models.py`: SQLAlchemy data model and relationships.
- `services/auth.py`: session identity and role guards.
- `services/ml.py`: model loading and fallback prediction.
- `ml/train_model.py`: reproducible training and evaluation.
- `templates/` and `static/`: server-rendered responsive UI.
- `tests/`: automated core tests.

## Database tables
- `users`: student/admin identities and active state.
- `complaints`: complaint details, predicted/final category, priority, assignment, status and timestamps.
- `complaint_status_history`: status changes and separate public/internal notes.
- `complaint_feedback`: one rating per complaint.

## Viva preparation
**Q: Why Flask?** Lightweight Python framework with simple routing and Jinja templates.
**Q: Why SQLAlchemy?** ORM models and parameterized database operations reduce SQL injection risk and simplify relationships.
**Q: How are passwords stored?** Werkzeug stores salted password hashes; raw passwords are not saved.
**Q: Why TF-IDF?** It represents text by weighted terms, reducing the impact of common words.
**Q: Why Multinomial Naive Bayes?** It is fast and a reasonable baseline for sparse text classification.
**Q: Why retain predicted and final categories?** To let students correct predictions and later compare model suggestions with human choices.
**Q: How are internal notes protected?** The student template renders only `public_comment`; internal notes appear only on admin pages.
**Q: What are the model limitations?** The sample dataset is small and synthetic, so its test score is only a demonstration.
**Q: How is a complaint protected from ID guessing?** Student routes verify the logged-in user's ID owns the complaint before displaying or changing it.
**Q: How is status workflow enforced?** Server-side transition rules reject invalid status changes.
