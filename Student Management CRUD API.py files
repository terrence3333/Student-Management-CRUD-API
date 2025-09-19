# Student Management CRUD API

A complete FastAPI application for managing students, courses, and enrollments with full CRUD operations.

## Project Structure
```
student-management-api/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── database.py
│   ├── models.py
│   ├── schemas.py
│   └── crud.py
├── requirements.txt
├── README.md
└── .env.example
```

## Files

### requirements.txt
```txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
python-dotenv==1.0.0
pydantic==2.5.0
```

### .env.example
```env
DATABASE_URL=postgresql://username:password@localhost:5432/student_management
SECRET_KEY=your-secret-key-here
```

### app/__init__.py
```python
# Empty file to make app a Python package
```

### app/database.py
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os
from dotenv import load_dotenv

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://postgres:password@localhost:5432/student_management")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### app/models.py
```python
from sqlalchemy import Column, Integer, String, DateTime, Text, ForeignKey, Table
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from .database import Base

# Association table for many-to-many relationship between students and courses
student_course_association = Table(
    'enrollments',
    Base.metadata,
    Column('id', Integer, primary_key=True),
    Column('student_id', Integer, ForeignKey('students.id'), nullable=False),
    Column('course_id', Integer, ForeignKey('courses.id'), nullable=False),
    Column('enrolled_at', DateTime(timezone=True), server_default=func.now()),
    Column('grade', String(2), nullable=True)  # A+, A, B+, B, C+, C, D, F
)

class Student(Base):
    __tablename__ = "students"

    id = Column(Integer, primary_key=True, index=True)
    student_id = Column(String(20), unique=True, index=True, nullable=False)
    first_name = Column(String(50), nullable=False)
    last_name = Column(String(50), nullable=False)
    email = Column(String(100), unique=True, index=True, nullable=False)
    phone = Column(String(15), nullable=True)
    date_of_birth = Column(DateTime, nullable=True)
    address = Column(Text, nullable=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    # Relationship to courses through enrollments
    courses = relationship("Course", secondary=student_course_association, back_populates="students")

class Course(Base):
    __tablename__ = "courses"

    id = Column(Integer, primary_key=True, index=True)
    course_code = Column(String(10), unique=True, index=True, nullable=False)
    course_name = Column(String(100), nullable=False)
    description = Column(Text, nullable=True)
    credits = Column(Integer, nullable=False, default=3)
    instructor = Column(String(100), nullable=True)
    semester = Column(String(20), nullable=True)  # Fall 2024, Spring 2025, etc.
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    # Relationship to students through enrollments
    students = relationship("Student", secondary=student_course_association, back_populates="courses")
```

### app/schemas.py
```python
from pydantic import BaseModel, EmailStr
from typing import Optional, List
from datetime import datetime

# Student Schemas
class StudentBase(BaseModel):
    student_id: str
    first_name: str
    last_name: str
    email: EmailStr
    phone: Optional[str] = None
    date_of_birth: Optional[datetime] = None
    address: Optional[str] = None

class StudentCreate(StudentBase):
    pass

class StudentUpdate(BaseModel):
    first_name: Optional[str] = None
    last_name: Optional[str] = None
    email: Optional[EmailStr] = None
    phone: Optional[str] = None
    date_of_birth: Optional[datetime] = None
    address: Optional[str] = None

class Student(StudentBase):
    id: int
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        from_attributes = True

class StudentWithCourses(Student):
    courses: List["CourseBasic"] = []

# Course Schemas
class CourseBase(BaseModel):
    course_code: str
    course_name: str
    description: Optional[str] = None
    credits: int = 3
    instructor: Optional[str] = None
    semester: Optional[str] = None

class CourseCreate(CourseBase):
    pass

class CourseUpdate(BaseModel):
    course_name: Optional[str] = None
    description: Optional[str] = None
    credits: Optional[int] = None
    instructor: Optional[str] = None
    semester: Optional[str] = None

class CourseBasic(BaseModel):
    id: int
    course_code: str
    course_name: str
    credits: int
    
    class Config:
        from_attributes = True

class Course(CourseBase):
    id: int
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        from_attributes = True

class CourseWithStudents(Course):
    students: List["Student"] = []

# Enrollment Schemas
class EnrollmentCreate(BaseModel):
    student_id: int
    course_id: int

class EnrollmentUpdate(BaseModel):
    grade: Optional[str] = None

class Enrollment(BaseModel):
    student_id: int
    course_id: int
    enrolled_at: datetime
    grade: Optional[str] = None
    
    class Config:
        from_attributes = True
```

### app/crud.py
```python
from sqlalchemy.orm import Session
from sqlalchemy import and_
from . import models, schemas
from typing import List, Optional

# Student CRUD operations
def get_student(db: Session, student_id: int):
    return db.query(models.Student).filter(models.Student.id == student_id).first()

def get_student_by_student_id(db: Session, student_id: str):
    return db.query(models.Student).filter(models.Student.student_id == student_id).first()

def get_student_by_email(db: Session, email: str):
    return db.query(models.Student).filter(models.Student.email == email).first()

def get_students(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.Student).offset(skip).limit(limit).all()

def create_student(db: Session, student: schemas.StudentCreate):
    db_student = models.Student(**student.dict())
    db.add(db_student)
    db.commit()
    db.refresh(db_student)
    return db_student

def update_student(db: Session, student_id: int, student_update: schemas.StudentUpdate):
    db_student = get_student(db, student_id)
    if db_student:
        update_data = student_update.dict(exclude_unset=True)
        for field, value in update_data.items():
            setattr(db_student, field, value)
        db.commit()
        db.refresh(db_student)
    return db_student

def delete_student(db: Session, student_id: int):
    db_student = get_student(db, student_id)
    if db_student:
        db.delete(db_student)
        db.commit()
    return db_student

# Course CRUD operations
def get_course(db: Session, course_id: int):
    return db.query(models.Course).filter(models.Course.id == course_id).first()

def get_course_by_code(db: Session, course_code: str):
    return db.query(models.Course).filter(models.Course.course_code == course_code).first()

def get_courses(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.Course).offset(skip).limit(limit).all()

def create_course(db: Session, course: schemas.CourseCreate):
    db_course = models.Course(**course.dict())
    db.add(db_course)
    db.commit()
    db.refresh(db_course)
    return db_course

def update_course(db: Session, course_id: int, course_update: schemas.CourseUpdate):
    db_course = get_course(db, course_id)
    if db_course:
        update_data = course_update.dict(exclude_unset=True)
        for field, value in update_data.items():
            setattr(db_course, field, value)
        db.commit()
        db.refresh(db_course)
    return db_course

def delete_course(db: Session, course_id: int):
    db_course = get_course(db, course_id)
    if db_course:
        db.delete(db_course)
        db.commit()
    return db_course

# Enrollment operations
def enroll_student(db: Session, student_id: int, course_id: int):
    # Check if student and course exist
    student = get_student(db, student_id)
    course = get_course(db, course_id)
    
    if not student or not course:
        return None
    
    # Check if already enrolled
    existing = db.execute(
        models.student_course_association.select().where(
            and_(
                models.student_course_association.c.student_id == student_id,
                models.student_course_association.c.course_id == course_id
            )
        )
    ).first()
    
    if existing:
        return None  # Already enrolled
    
    # Create enrollment
    db.execute(
        models.student_course_association.insert().values(
            student_id=student_id,
            course_id=course_id
        )
    )
    db.commit()
    return True

def unenroll_student(db: Session, student_id: int, course_id: int):
    result = db.execute(
        models.student_course_association.delete().where(
            and_(
                models.student_course_association.c.student_id == student_id,
                models.student_course_association.c.course_id == course_id
            )
        )
    )
    db.commit()
    return result.rowcount > 0

def get_student_enrollments(db: Session, student_id: int):
    student = db.query(models.Student).filter(models.Student.id == student_id).first()
    return student.courses if student else []

def get_course_enrollments(db: Session, course_id: int):
    course = db.query(models.Course).filter(models.Course.id == course_id).first()
    return course.students if course else []

def update_grade(db: Session, student_id: int, course_id: int, grade: str):
    db.execute(
        models.student_course_association.update().where(
            and_(
                models.student_course_association.c.student_id == student_id,
                models.student_course_association.c.course_id == course_id
            )
        ).values(grade=grade)
    )
    db.commit()
    return True
```

### app/main.py
```python
from fastapi import FastAPI, Depends, HTTPException, status
from sqlalchemy.orm import Session
from typing import List
from . import crud, models, schemas
from .database import SessionLocal, engine, get_db

# Create database tables
models.Base.metadata.create_all(bind=engine)

app = FastAPI(
    title="Student Management API",
    description="A CRUD API for managing students, courses, and enrollments",
    version="1.0.0"
)

# Student endpoints
@app.post("/students/", response_model=schemas.Student, status_code=status.HTTP_201_CREATED)
def create_student(student: schemas.StudentCreate, db: Session = Depends(get_db)):
    # Check if student ID or email already exists
    db_student = crud.get_student_by_student_id(db, student_id=student.student_id)
    if db_student:
        raise HTTPException(status_code=400, detail="Student ID already registered")
    
    db_student = crud.get_student_by_email(db, email=student.email)
    if db_student:
        raise HTTPException(status_code=400, detail="Email already registered")
    
    return crud.create_student(db=db, student=student)

@app.get("/students/", response_model=List[schemas.Student])
def read_students(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    students = crud.get_students(db, skip=skip, limit=limit)
    return students

@app.get("/students/{student_id}", response_model=schemas.StudentWithCourses)
def read_student(student_id: int, db: Session = Depends(get_db)):
    db_student = crud.get_student(db, student_id=student_id)
    if db_student is None:
        raise HTTPException(status_code=404, detail="Student not found")
    return db_student

@app.put("/students/{student_id}", response_model=schemas.Student)
def update_student(student_id: int, student_update: schemas.StudentUpdate, db: Session = Depends(get_db)):
    db_student = crud.update_student(db, student_id=student_id, student_update=student_update)
    if db_student is None:
        raise HTTPException(status_code=404, detail="Student not found")
    return db_student

@app.delete("/students/{student_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_student(student_id: int, db: Session = Depends(get_db)):
    db_student = crud.delete_student(db, student_id=student_id)
    if db_student is None:
        raise HTTPException(status_code=404, detail="Student not found")

# Course endpoints
@app.post("/courses/", response_model=schemas.Course, status_code=status.HTTP_201_CREATED)
def create_course(course: schemas.CourseCreate, db: Session = Depends(get_db)):
    db_course = crud.get_course_by_code(db, course_code=course.course_code)
    if db_course:
        raise HTTPException(status_code=400, detail="Course code already exists")
    
    return crud.create_course(db=db, course=course)

@app.get("/courses/", response_model=List[schemas.Course])
def read_courses(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    courses = crud.get_courses(db, skip=skip, limit=limit)
    return courses

@app.get("/courses/{course_id}", response_model=schemas.CourseWithStudents)
def read_course(course_id: int, db: Session = Depends(get_db)):
    db_course = crud.get_course(db, course_id=course_id)
    if db_course is None:
        raise HTTPException(status_code=404, detail="Course not found")
    return db_course

@app.put("/courses/{course_id}", response_model=schemas.Course)
def update_course(course_id: int, course_update: schemas.CourseUpdate, db: Session = Depends(get_db)):
    db_course = crud.update_course(db, course_id=course_id, course_update=course_update)
    if db_course is None:
        raise HTTPException(status_code=404, detail="Course not found")
    return db_course

@app.delete("/courses/{course_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_course(course_id: int, db: Session = Depends(get_db)):
    db_course = crud.delete_course(db, course_id=course_id)
    if db_course is None:
        raise HTTPException(status_code=404, detail="Course not found")

# Enrollment endpoints
@app.post("/enrollments/", status_code=status.HTTP_201_CREATED)
def enroll_student(enrollment: schemas.EnrollmentCreate, db: Session = Depends(get_db)):
    result = crud.enroll_student(db, enrollment.student_id, enrollment.course_id)
    if result is None:
        raise HTTPException(status_code=400, detail="Student or course not found, or already enrolled")
    return {"message": "Student enrolled successfully"}

@app.delete("/enrollments/", status_code=status.HTTP_204_NO_CONTENT)
def unenroll_student(enrollment: schemas.EnrollmentCreate, db: Session = Depends(get_db)):
    result = crud.unenroll_student(db, enrollment.student_id, enrollment.course_id)
    if not result:
        raise HTTPException(status_code=404, detail="Enrollment not found")

@app.get("/students/{student_id}/courses", response_model=List[schemas.CourseBasic])
def get_student_courses(student_id: int, db: Session = Depends(get_db)):
    courses = crud.get_student_enrollments(db, student_id)
    if courses is None:
        raise HTTPException(status_code=404, detail="Student not found")
    return courses

@app.get("/courses/{course_id}/students", response_model=List[schemas.Student])
def get_course_students(course_id: int, db: Session = Depends(get_db)):
    students = crud.get_course_enrollments(db, course_id)
    if students is None:
        raise HTTPException(status_code=404, detail="Course not found")
    return students

@app.put("/enrollments/{student_id}/{course_id}/grade")
def update_student_grade(student_id: int, course_id: int, grade_update: schemas.EnrollmentUpdate, db: Session = Depends(get_db)):
    if grade_update.grade:
        crud.update_grade(db, student_id, course_id, grade_update.grade)
        return {"message": "Grade updated successfully"}
    raise HTTPException(status_code=400, detail="Grade is required")

@app.get("/")
def root():
    return {
        "message": "Welcome to Student Management API",
        "docs": "/docs",
        "redoc": "/redoc"
    }

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Setup Instructions

### 1. Clone and Setup
```bash
git clone <your-repo-url>
cd student-management-api
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Database Setup
```bash
# Install PostgreSQL and create database
createdb student_management

# Copy environment file and configure
cp .env.example .env
# Edit .env with your database credentials
```

### 3. Run the Application
```bash
cd app
uvicorn main:app --reload
```

The API will be available at:
- **Application**: http://localhost:8000
- **Interactive Docs**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

## API Endpoints

### Students
- `POST /students/` - Create a new student
- `GET /students/` - Get all students (with pagination)
- `GET /students/{student_id}` - Get student by ID (with enrolled courses)
- `PUT /students/{student_id}` - Update student
- `DELETE /students/{student_id}` - Delete student

### Courses
- `POST /courses/` - Create a new course
- `GET /courses/` - Get all courses (with pagination)
- `GET /courses/{course_id}` - Get course by ID (with enrolled students)
- `PUT /courses/{course_id}` - Update course
- `DELETE /courses/{course_id}` - Delete course

### Enrollments
- `POST /enrollments/` - Enroll student in course
- `DELETE /enrollments/` - Unenroll student from course
- `GET /students/{student_id}/courses` - Get courses for a student
- `GET /courses/{course_id}/students` - Get students in a course
- `PUT /enrollments/{student_id}/{course_id}/grade` - Update student's grade

## Example Usage

### Create a Student
```bash
curl -X POST "http://localhost:8000/students/" \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "STU001",
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@email.com",
    "phone": "+1234567890"
  }'
```

### Create a Course
```bash
curl -X POST "http://localhost:8000/courses/" \
  -H "Content-Type: application/json" \
  -d '{
    "course_code": "CS101",
    "course_name": "Introduction to Computer Science",
    "description": "Basic concepts of computer science",
    "credits": 3,
    "instructor": "Dr. Smith",
    "semester": "Fall 2024"
  }'
```

### Enroll Student in Course
```bash
curl -X POST "http://localhost:8000/enrollments/" \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": 1,
    "course_id": 1
  }'
```

## Features

- **Complete CRUD operations** for Students and Courses
- **Many-to-many relationship** through enrollments
- **Data validation** using Pydantic schemas
- **Automatic API documentation** with FastAPI
- **Database relationships** with SQLAlchemy ORM
- **Error handling** with proper HTTP status codes
- **Pagination support** for list endpoints
- **Grade management** for enrollments

## Database Schema

The application uses three main tables:
1. **students** - Student information
2. **courses** - Course information  
3. **enrollments** - Many-to-many relationship table with grades

## Testing

Visit http://localhost:8000/docs for interactive API testing with Swagger UI.
