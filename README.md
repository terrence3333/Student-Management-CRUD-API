# Student Management CRUD API

A comprehensive FastAPI-based REST API for managing students, courses, and enrollments in an educational institution.

## 📋 Table of Contents

- [Features](#features)
- [Technology Stack](#technology-stack)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Database Configuration](#database-configuration)
- [Running the Application](#running-the-application)
- [API Documentation](#api-documentation)
- [API Endpoints](#api-endpoints)
- [Usage Examples](#usage-examples)
- [Testing](#testing)
- [Project Structure](#project-structure)
- [Contributing](#contributing)

## ✨ Features

- **Complete CRUD Operations** for Students and Courses
- **Many-to-Many Relationships** through enrollments
- **Grade Management** for student-course enrollments
- **Data Validation** using Pydantic schemas
- **Automatic API Documentation** with Swagger UI
- **Database Relationships** with SQLAlchemy ORM
- **Error Handling** with proper HTTP status codes
- **Pagination Support** for list endpoints
- **Email and Student ID Uniqueness** validation

## 🛠 Technology Stack

- **FastAPI** - Modern, fast web framework for building APIs
- **SQLAlchemy** - SQL toolkit and Object-Relational Mapping (ORM)
- **PostgreSQL** - Advanced open source relational database
- **Pydantic** - Data validation and settings management using Python type hints
- **Uvicorn** - Lightning-fast ASGI server

## 📋 Prerequisites

Before running this application, make sure you have the following installed:

- **Python 3.8+** ([Download Python](https://www.python.org/downloads/))
- **PostgreSQL** ([Download PostgreSQL](https://www.postgresql.org/download/))
- **Git** ([Download Git](https://git-scm.com/downloads/))

## 🚀 Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/student-management-api.git
cd student-management-api
```

### 2. Create Virtual Environment

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate

# On Windows:
venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Environment Configuration

```bash
# Copy the example environment file
cp .env.example .env

# Edit the .env file with your database credentials
nano .env  # or use your preferred editor
```

## 🗄️ Database Configuration

### 1. Install PostgreSQL

Follow the installation guide for your operating system:
- **macOS**: Use Homebrew: `brew install postgresql`
- **Ubuntu/Debian**: `sudo apt-get install postgresql postgresql-contrib`
- **Windows**: Download from [PostgreSQL official site](https://www.postgresql.org/download/windows/)

### 2. Create Database

```bash
# Connect to PostgreSQL
psql -U postgres

# Create database
CREATE DATABASE student_management;

# Create user (optional)
CREATE USER student_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE student_management TO student_user;

# Exit PostgreSQL
\q
```

### 3. Configure Environment Variables

Edit your `.env` file:

```env
DATABASE_URL=postgresql://username:password@localhost:5432/student_management
SECRET_KEY=your-secret-key-here-change-this-in-production
```

**Example configurations:**

```env
# Using postgres user
DATABASE_URL=postgresql://postgres:password@localhost:5432/student_management

# Using custom user
DATABASE_URL=postgresql://student_user:your_password@localhost:5432/student_management

# Using cloud database (example with Heroku)
DATABASE_URL=postgresql://username:password@host:port/database_name
```

## ▶️ Running the Application

### 1. Start the Development Server

```bash
# Navigate to the app directory
cd app

# Run the application
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### 2. Alternative: Run from Root Directory

```bash
# From project root
uvicorn app.main:app --reload
```

### 3. Production Deployment

```bash
# For production (without reload)
uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

## 📚 API Documentation

Once the application is running, you can access the interactive API documentation:

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc
- **OpenAPI JSON**: http://localhost:8000/openapi.json

## 🔗 API Endpoints

### Root Endpoint
```
GET /
```
Returns welcome message and documentation links.

### Student Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/students/` | Create a new student |
| `GET` | `/students/` | Get all students (paginated) |
| `GET` | `/students/{student_id}` | Get student by ID with courses |
| `PUT` | `/students/{student_id}` | Update student information |
| `DELETE` | `/students/{student_id}` | Delete a student |
| `GET` | `/students/{student_id}/courses` | Get all courses for a student |

### Course Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/courses/` | Create a new course |
| `GET` | `/courses/` | Get all courses (paginated) |
| `GET` | `/courses/{course_id}` | Get course by ID with students |
| `PUT` | `/courses/{course_id}` | Update course information |
| `DELETE` | `/courses/{course_id}` | Delete a course |
| `GET` | `/courses/{course_id}/students` | Get all students in a course |

### Enrollment Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/enrollments/` | Enroll student in a course |
| `DELETE` | `/enrollments/` | Remove student from a course |
| `PUT` | `/enrollments/{student_id}/{course_id}/grade` | Update student's grade |

## 📝 Usage Examples

### 1. Create a Student

```bash
curl -X POST "http://localhost:8000/students/" \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "STU001",
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@university.edu",
    "phone": "+1234567890",
    "address": "123 College St, University City, UC 12345"
  }'
```

**Response:**
```json
{
  "id": 1,
  "student_id": "STU001",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@university.edu",
  "phone": "+1234567890",
  "date_of_birth": null,
  "address": "123 College St, University City, UC 12345",
  "created_at": "2024-01-15T10:30:00.123456Z",
  "updated_at": null
}
```

### 2. Create a Course

```bash
curl -X POST "http://localhost:8000/courses/" \
  -H "Content-Type: application/json" \
  -d '{
    "course_code": "CS101",
    "course_name": "Introduction to Computer Science",
    "description": "Fundamental concepts of computer science including algorithms, data structures, and programming.",
    "credits": 3,
    "instructor": "Dr. Jane Smith",
    "semester": "Fall 2024"
  }'
```

### 3. Enroll Student in Course

```bash
curl -X POST "http://localhost:8000/enrollments/" \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": 1,
    "course_id": 1
  }'
```

### 4. Update Student Information

```bash
curl -X PUT "http://localhost:8000/students/1" \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "+1987654321",
    "address": "456 New Address St, Updated City, UC 54321"
  }'
```

### 5. Get Student with Enrolled Courses

```bash
curl -X GET "http://localhost:8000/students/1"
```

**Response:**
```json
{
  "id": 1,
  "student_id": "STU001",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@university.edu",
  "phone": "+1987654321",
  "date_of_birth": null,
  "address": "456 New Address St, Updated City, UC 54321",
  "created_at": "2024-01-15T10:30:00.123456Z",
  "updated_at": "2024-01-15T11:15:30.789012Z",
  "courses": [
    {
      "id": 1,
      "course_code": "CS101",
      "course_name": "Introduction to Computer Science",
      "credits": 3
    }
  ]
}
```

### 6. Update Student's Grade

```bash
curl -X PUT "http://localhost:8000/enrollments/1/1/grade" \
  -H "Content-Type: application/json" \
  -d '{
    "grade": "A"
  }'
```

### 7. Get All Students (with Pagination)

```bash
curl -X GET "http://localhost:8000/students/?skip=0&limit=10"
```

### 8. Delete a Student

```bash
curl -X DELETE "http://localhost:8000/students/1"
```

## 🧪 Testing

### 1. Interactive Testing

Visit http://localhost:8000/docs to use the interactive Swagger UI for testing all endpoints.

### 2. Manual Testing Steps

1. **Create a student** using the POST `/students/` endpoint
2. **Create a course** using the POST `/courses/` endpoint
3. **Enroll the student** in the course using POST `/enrollments/`
4. **View the student** with enrolled courses using GET `/students/{id}`
5. **Update the student's grade** using PUT `/enrollments/{student_id}/{course_id}/grade`
6. **Test all other endpoints** as needed

### 3. Sample Test Data

**Student Data:**
```json
{
  "student_id": "STU001",
  "first_name": "Alice",
  "last_name": "Johnson",
  "email": "alice.johnson@university.edu",
  "phone": "+1234567890"
}
```

**Course Data:**
```json
{
  "course_code": "MATH101",
  "course_name": "Calculus I",
  "description": "Introduction to differential and integral calculus",
  "credits": 4,
  "instructor": "Prof. Robert Brown",
  "semester": "Spring 2024"
}
```

## 📁 Project Structure

```
student-management-api/
├── app/
│   ├── __init__.py          # Package initialization
│   ├── main.py              # FastAPI application and routes
│   ├── database.py          # Database connection and session management
│   ├── models.py            # SQLAlchemy database models
│   ├── schemas.py           # Pydantic schemas for request/response
│   └── crud.py              # Database CRUD operations
├── requirements.txt         # Python dependencies
├── .env.example            # Environment variables template
├── .gitignore              # Git ignore rules
├── README.md               # Project documentation
└── postman_collection.json # Postman API collection
```

## 🔧 Configuration Options

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | Required |
| `SECRET_KEY` | Secret key for security | Required |

### Query Parameters

**Pagination (for list endpoints):**
- `skip`: Number of records to skip (default: 0)
- `limit`: Maximum number of records to return (default: 100)

## 🚨 Error Handling

The API returns appropriate HTTP status codes:

| Status Code | Description |
|-------------|-------------|
| `200` | Success |
| `201` | Created successfully |
| `204` | No content (successful deletion) |
| `400` | Bad request (validation error) |
| `404` | Resource not found |
| `422` | Unprocessable entity (validation error) |
| `500` | Internal server error |

**Example Error Response:**
```json
{
  "detail": "Student ID already registered"
}
```

## 🔒 Data Validation

The API enforces the following validation rules:

**Students:**
- Student ID must be unique
- Email must be unique and valid format
- First name and last name are required
- Phone number is optional

**Courses:**
- Course code must be unique
- Course name is required
- Credits must be a positive integer

**Enrollments:**
- Student and course must exist
- Cannot enroll the same student in the same course twice
- Grades must follow standard format (A+, A, B+, B, C+, C, D, F)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-feature`)
3. Make your changes
4. Add tests if applicable
5. Commit your changes (`git commit -am 'Add new feature'`)
6. Push to the branch (`git push origin feature/new-feature`)
7. Create a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 💡 Additional Notes

### Database Migrations

For production use, consider implementing database migrations using Alembic:

```bash
pip install alembic
alembic init alembic
alembic revision --autogenerate -m "Initial migration"
alembic upgrade head
```

### Security Considerations

For production deployment:
- Use environment variables for sensitive data
- Implement authentication and authorization
- Use HTTPS
- Set up proper CORS policies
- Validate and sanitize all inputs

### Performance Optimization

- Implement database indexing
- Use connection pooling
- Add caching for frequently accessed data
- Implement API rate limiting

---

**Happy Coding! 🚀**

For questions or support, please open an issue in the GitHub repository.
