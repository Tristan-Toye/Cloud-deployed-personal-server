# IDentiGate Technical Documentation

## Table of Contents

1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Database Schema](#database-schema)
4. [Authentication Flow](#authentication-flow)
5. [API Reference](#api-reference)
6. [Configuration](#configuration)
7. [Security Implementation](#security-implementation)
8. [Deployment Guide](#deployment-guide)
9. [Troubleshooting](#troubleshooting)
10. [Development Guidelines](#development-guidelines)

## Overview

IDentiGate is a multi-factor authentication system designed for secure access control in organizational environments. The system combines traditional password authentication with biometric face recognition and time-based one-time passwords (TOTP) to provide robust security.

### Key Components

- **Flask Web Application**: Main application framework
- **PostgreSQL Database**: Data persistence layer
- **Face Recognition Engine**: Biometric authentication
- **WebSocket Communication**: Real-time face processing
- **QR Code System**: Access token generation and validation

### System Requirements

- **Python**: 3.8 or higher
- **Database**: PostgreSQL 12+
- **Memory**: Minimum 512MB RAM
- **Storage**: 1GB available space
- **Network**: HTTPS support for production

## System Architecture

### High-Level Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web Browser   │    │   Mobile App    │    │   QR Scanner    │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌─────────────▼─────────────┐
                    │      Flask Application    │
                    │  (IDentiGate Web Server)  │
                    └─────────────┬─────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │    PostgreSQL Database    │
                    │   (User Data & Logs)      │
                    └───────────────────────────┘
```

### Application Structure

```
flask_info/
├── __init__.py                 # Flask app initialization
├── models.py                   # Database models
├── routes_functions.py         # Route handlers
├── forms.py                    # Form definitions
├── functions.py                # Utility functions
├── decorators.py               # Custom decorators
├── live_face_regconition.py    # Face recognition logic
├── save_faces.py               # Face data management
├── constants.py                # Configuration constants
└── templates_with_css/         # Frontend assets
    ├── *.html                  # HTML templates
    ├── css/                    # Stylesheets
    ├── js/                     # JavaScript files
    └── images/                 # Static images
```

## Database Schema

### Core Tables

#### Users Table
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username VARCHAR NOT NULL,
    national_number VARCHAR NOT NULL UNIQUE,
    email_address VARCHAR NOT NULL UNIQUE,
    password VARCHAR NOT NULL,
    faces BYTEA,  -- PickleType for face encodings
    qr_leave VARCHAR NOT NULL UNIQUE,
    qr_leave_code VARCHAR NOT NULL UNIQUE
);
```

#### Roles Table
```sql
CREATE TABLE roles (
    id INTEGER PRIMARY KEY,
    name VARCHAR NOT NULL UNIQUE
);
```

#### User Roles (Junction Table)
```sql
CREATE TABLE user_roles (
    id INTEGER PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    role_id INTEGER REFERENCES roles(id) ON DELETE CASCADE
);
```

#### Logs Table
```sql
CREATE TABLE logs (
    id INTEGER PRIMARY KEY,
    date_entry TIMESTAMP NOT NULL DEFAULT NOW(),
    date_exit TIMESTAMP NULL
);
```

#### User Logs (Junction Table)
```sql
CREATE TABLE user_logs (
    id INTEGER PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    log_id INTEGER REFERENCES logs(id) ON DELETE CASCADE
);
```

#### QR Codes Table
```sql
CREATE TABLE qr_codes (
    id INTEGER PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL DEFAULT NOW(),
    code VARCHAR NOT NULL UNIQUE
);
```

#### User QR (Junction Table)
```sql
CREATE TABLE user_qr (
    id INTEGER PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    qr_id INTEGER REFERENCES qr_codes(id) ON DELETE CASCADE
);
```

#### QR Visitor Table
```sql
CREATE TABLE qr_visitor (
    id INTEGER PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL DEFAULT NOW(),
    code VARCHAR NOT NULL UNIQUE,
    company VARCHAR NOT NULL
);
```

## Authentication Flow

### Multi-Factor Authentication Process

#### 1. Password Authentication
```python
# Route: /home/login
def login():
    # 1. Validate email/password combination
    # 2. Check user role
    # 3. Redirect to appropriate MFA step
```

#### 2. Role-Based Authentication Requirements

| Role | Authentication Steps |
|------|---------------------|
| Staff | Password only |
| Security | Password + TOTP |
| Recruiter | Password + TOTP |
| Admin | Password + Face Recognition + TOTP |

#### 3. Face Recognition Process
```python
# WebSocket event: 'stream'
@socketio.on('stream')
def stream(data_image):
    # 1. Decode base64 image
    # 2. Detect face locations
    # 3. Generate face encodings
    # 4. Compare with stored encodings
    # 5. Return match result
```

#### 4. TOTP Verification
```python
# Route: /home/time_based_authentication
def time_based_authentication():
    # 1. Validate TOTP code
    # 2. Check attempt limits
    # 3. Complete login process
```

## API Reference

### Authentication Endpoints

#### POST /home/login
Authenticate user with email and password.

**Request Body:**
```json
{
    "email_address": "user@example.com",
    "password": "userpassword"
}
```

**Response:**
- `200 OK`: Redirect to appropriate MFA step
- `401 Unauthorized`: Invalid credentials

#### POST /home/time_based_authentication
Verify TOTP code for multi-factor authentication.

**Request Body:**
```json
{
    "time_based_pincode": "123456"
}
```

**Response:**
- `200 OK`: Authentication successful
- `401 Unauthorized`: Invalid TOTP code

### User Management Endpoints

#### POST /home/register_staff
Register new staff member (Admin/Recruiter only).

**Request Body:**
```json
{
    "username": "john_doe",
    "email_address": "john@example.com",
    "password1": "password123",
    "password2": "password123",
    "national_number": "123456789"
}
```

#### POST /home/register_employee
Register new employee with role assignment (Admin only).

**Request Body:**
```json
{
    "username": "jane_smith",
    "email_address": "jane@example.com",
    "password1": "password123",
    "password2": "password123",
    "national_number": "987654321",
    "role": "security"
}
```

### QR Code Endpoints

#### POST /home/QR_code_request
Generate entry QR code for authenticated user.

**Request Body:**
```json
{
    "password": "userpassword"
}
```

**Response:**
```json
{
    "qr_code": "base64_encoded_image",
    "expires_at": "2024-01-01T12:00:00Z"
}
```

#### GET /home/QR_code_leave
Generate leave QR code for authenticated user.

**Response:**
```json
{
    "qr_code": "unique_leave_code",
    "user_id": 123
}
```

### Monitoring Endpoints

#### GET /home/employee_list
Get list of all employees (Admin/Security only).

**Response:**
```json
[
    {
        "id": 1,
        "username": "john_doe",
        "email": "john@example.com",
        "role": "staff",
        "last_login": "2024-01-01T10:00:00Z"
    }
]
```

#### GET /home/employee_list/{national_number}
Get detailed logs for specific employee.

**Response:**
```json
{
    "user": {
        "username": "john_doe",
        "role": "staff"
    },
    "logs": [
        {
            "date_entry": "2024-01-01T09:00:00Z",
            "date_exit": "2024-01-01T17:00:00Z"
        }
    ]
}
```

### WebSocket Events

#### Client to Server

- `stream`: Send face recognition image data
- `stream_register_faces`: Send face registration image data
- `stream_remove_employee`: Send face verification for employee removal

#### Server to Client

- `face_recognition`: Face recognition result
- `register_face`: Face registration result
- `response_remove_employee`: Employee removal verification result

## Configuration

### Environment Variables

```bash
# Flask Configuration
FLASK_APP=app.py
FLASK_ENV=development
SECRET_KEY=your_secret_key_here

# Database Configuration
DATABASE_URL=postgresql://user:password@host:port/database

# Email Configuration (if enabled)
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=465
MAIL_USE_SSL=True
MAIL_USERNAME=your_email@gmail.com
MAIL_PASSWORD=your_app_password
```

### Application Constants

```python
# flask_info/constants.py

# Face Recognition Settings
MODEL = 'hog'                    # 'hog' or 'cnn'
TOLERANCE = 0.6                  # Face matching tolerance (0.0-1.0)

# Authentication Settings
max_attempts_google_auth = 3     # Maximum TOTP attempts
max_attempts_password = 5        # Maximum password attempts
min_attempts_password_before_notification = 2

# QR Code Settings
time_interval = 43200            # QR code validity in seconds (12 hours)
kenteken = "}"                   # QR code delimiter

# Security Settings
SALT = b'your_salt_here'         # Encryption salt
cutoff_hash = 32                 # Hash length

# TOTP Settings
secret_key_pyotp = 'your_totp_secret_key'
```

## Security Implementation

### Password Security

```python
# Password hashing using Flask-User
from flask_user import UserManager

user_manager = UserManager(app, db, User)
hashed_password = user_manager.hash_password(plain_password)
is_valid = user_manager.verify_password(plain_password, hashed_password)
```

### Face Data Encryption

```python
# Face encodings stored as encrypted pickle objects
import pickle
from cryptography.fernet import Fernet

def encrypt_face_data(face_encodings):
    key = Fernet.generate_key()
    cipher = Fernet(key)
    pickled_data = pickle.dumps(face_encodings)
    encrypted_data = cipher.encrypt(pickled_data)
    return encrypted_data, key
```

### Session Security

```python
# Secure session configuration
app.config['SESSION_TYPE'] = 'sqlalchemy'
app.config['SESSION_SQLALCHEMY_TABLE'] = 'session_sqlalchemy'
app.config['SESSION_COOKIE_SECURE'] = True
app.config['SESSION_COOKIE_HTTPONLY'] = True
```

### Rate Limiting

```python
# Implementation in decorators
def rate_limit(max_attempts, window_seconds):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            # Check attempt count in session
            # Block if limit exceeded
            return f(*args, **kwargs)
        return wrapper
    return decorator
```

## Deployment Guide

### Local Development Setup

1. **Install Dependencies**
   ```bash
   pip install -r requirements.txt
   ```

2. **Database Setup**
   ```bash
   # Create PostgreSQL database
   createdb identigate_dev
   
   # Set environment variable
   export DATABASE_URL=postgresql://localhost/identigate_dev
   ```

3. **Initialize Database**
   ```bash
   flask db init
   flask db migrate -m "Initial migration"
   flask db upgrade
   ```

4. **Create Admin User**
   ```python
   from flask_info import app, db
   from flask_info.models import User, Role
   
   with app.app_context():
       # Create admin role
       admin_role = Role(name='admin')
       db.session.add(admin_role)
       
       # Create admin user
       admin_user = User(
           username='admin',
           email_address='admin@example.com',
           password=user_manager.hash_password('admin123'),
           national_number='123456789'
       )
       admin_user.roles.append(admin_role)
       db.session.add(admin_user)
       db.session.commit()
   ```

### Production Deployment (Heroku)

1. **Create Heroku App**
   ```bash
   heroku create your-app-name
   ```

2. **Add PostgreSQL**
   ```bash
   heroku addons:create heroku-postgresql:hobby-dev
   ```

3. **Configure Environment**
   ```bash
   heroku config:set SECRET_KEY=$(python -c 'import secrets; print(secrets.token_hex(32))')
   heroku config:set PYTHONPATH=/app
   ```

4. **Deploy Application**
   ```bash
   git push heroku main
   ```

5. **Run Migrations**
   ```bash
   heroku run flask db upgrade
   ```

### Docker Deployment

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["gunicorn", "-w", "1", "--threads", "100", "app:app"]
```

## Troubleshooting

### Common Issues

#### Face Recognition Not Working
```bash
# Check OpenCV installation
python -c "import cv2; print(cv2.__version__)"

# Verify webcam access
python -c "import cv2; cap = cv2.VideoCapture(0); print(cap.isOpened())"
```

#### Database Connection Issues
```bash
# Test database connection
python -c "
from flask_info import app, db
with app.app_context():
    try:
        db.engine.execute('SELECT 1')
        print('Database connection successful')
    except Exception as e:
        print(f'Database connection failed: {e}')
"
```

#### WebSocket Connection Problems
```javascript
// Check WebSocket connection in browser console
console.log(socket.connected);
socket.on('connect', () => console.log('Connected'));
socket.on('disconnect', () => console.log('Disconnected'));
```

### Performance Optimization

#### Database Optimization
```sql
-- Create indexes for frequently queried columns
CREATE INDEX idx_users_email ON users(email_address);
CREATE INDEX idx_logs_date_entry ON logs(date_entry);
CREATE INDEX idx_qr_codes_timestamp ON qr_codes(timestamp);
```

#### Face Recognition Optimization
```python
# Use CNN model for better accuracy (requires GPU)
MODEL = 'cnn'  # Instead of 'hog'

# Adjust tolerance for balance between security and usability
TOLERANCE = 0.5  # More strict matching
```

### Logging and Monitoring

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('identigate.log'),
        logging.StreamHandler()
    ]
)

# Log authentication attempts
logger = logging.getLogger(__name__)
logger.info(f"Login attempt for user: {email_address}")
```

## Development Guidelines

### Code Style

- Follow PEP 8 Python style guide
- Use type hints for function parameters
- Document all public functions and classes
- Keep functions small and focused

### Testing

```python
# Example test structure
import unittest
from flask_info import app, db
from flask_info.models import User, Role

class TestAuthentication(unittest.TestCase):
    def setUp(self):
        app.config['TESTING'] = True
        app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
        self.app = app.test_client()
        
    def test_user_login(self):
        # Test login functionality
        pass
```

### Security Best Practices

1. **Input Validation**: Always validate and sanitize user input
2. **SQL Injection Prevention**: Use SQLAlchemy ORM, never raw SQL
3. **XSS Prevention**: Escape output in templates
4. **CSRF Protection**: Use Flask-WTF CSRF tokens
5. **Secure Headers**: Implement security headers

### Performance Considerations

1. **Database Queries**: Use eager loading for relationships
2. **Caching**: Implement Redis for session storage
3. **Face Recognition**: Optimize image processing pipeline
4. **WebSocket**: Limit concurrent connections

---

*This documentation is maintained by the IDentiGate development team. For questions or contributions, please contact the team or create an issue in the repository.* 