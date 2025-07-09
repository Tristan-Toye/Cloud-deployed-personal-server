# IDentiGate - Multi-Factor Authentication System

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Flask](https://img.shields.io/badge/Flask-2.0.2-green.svg)](https://flask.palletsprojects.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

IDentiGate is a sophisticated multi-factor authentication system that combines face recognition, QR code generation, and time-based authentication to provide secure access control for organizations. Built with Flask and deployed on Heroku, it offers a comprehensive solution for employee and visitor management.

## 🚀 Features

### 🔐 Multi-Factor Authentication
- **Password Authentication**: Traditional email/password login
- **Face Recognition**: Real-time facial recognition using OpenCV and face_recognition library
- **Time-Based Authentication**: Google Authenticator integration with TOTP
- **Role-Based Access Control**: Different authentication levels for different user roles

### 👥 User Management
- **Employee Registration**: Admin can register new employees with different roles
- **Face Registration**: Secure face enrollment for admin users (5 face samples required)
- **Role Management**: Support for staff, security, recruiter, and admin roles
- **Employee Directory**: View and manage all registered employees

### 📱 QR Code System
- **Entry QR Codes**: Generate time-limited QR codes for building entry
- **Leave QR Codes**: Unique QR codes for tracking employee exits
- **Visitor QR Codes**: Temporary QR codes for visitor access
- **QR Code Validation**: Secure validation with expiration times

### 📊 Monitoring & Logging
- **Entry/Exit Logging**: Track all employee movements
- **Real-time Monitoring**: Live view of present employees
- **Employee History**: Detailed logs for each employee
- **Analytics Dashboard**: Visual representation of attendance data

### 🛡️ Security Features
- **Encrypted Storage**: All sensitive data is encrypted before storage
- **Session Management**: Secure session handling with SQLAlchemy
- **Rate Limiting**: Protection against brute force attacks
- **WebSocket Communication**: Real-time face recognition processing

## 🏗️ Architecture

```
IDentiGate/
├── app.py                          # Main application entry point
├── flask_info/                     # Core application package
│   ├── __init__.py                # Flask app configuration
│   ├── models.py                  # Database models
│   ├── routes_functions.py        # Route handlers
│   ├── forms.py                   # WTForms definitions
│   ├── functions.py               # Utility functions
│   ├── decorators.py              # Custom decorators
│   ├── live_face_regconition.py   # Face recognition logic
│   ├── save_faces.py              # Face data management
│   └── constants.py               # Configuration constants
├── templates_with_css/            # HTML templates and static assets
│   ├── css/                       # Stylesheets
│   ├── js/                        # JavaScript files
│   └── images/                    # Static images
├── requirements.txt               # Python dependencies
├── Procfile                       # Heroku deployment configuration
└── runtime.txt                    # Python runtime specification
```

## 🛠️ Technology Stack

- **Backend**: Flask 2.0.2, SQLAlchemy, Flask-SocketIO
- **Database**: PostgreSQL (Heroku Postgres)
- **Authentication**: Flask-User, pyotp (TOTP)
- **Face Recognition**: face_recognition, OpenCV
- **Frontend**: HTML5, CSS3, JavaScript, WebSocket
- **Deployment**: Heroku, Gunicorn
- **QR Codes**: qrcode library
- **Data Visualization**: Plotly

## 📋 Prerequisites

- Python 3.8 or higher
- PostgreSQL database
- Webcam access (for face recognition)
- Google Authenticator app (for TOTP)

## 🚀 Installation

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/Cloud-deployed-personal-server.git
   cd Cloud-deployed-personal-server
   ```

2. **Create virtual environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure environment variables**
   ```bash
   # Create a .env file with your configuration
   export FLASK_APP=app.py
   export FLASK_ENV=development
   export DATABASE_URL=your_postgresql_connection_string
   export SECRET_KEY=your_secret_key
   ```

5. **Initialize database**
   ```bash
   flask db init
   flask db migrate
   flask db upgrade
   ```

6. **Run the application**
   ```bash
   python app.py
   ```

### Heroku Deployment

1. **Install Heroku CLI**
   ```bash
   # Follow instructions at https://devcenter.heroku.com/articles/heroku-cli
   ```

2. **Deploy to Heroku**
   ```bash
   heroku create your-app-name
   heroku addons:create heroku-postgresql:hobby-dev
   git push heroku main
   ```

3. **Configure environment variables**
   ```bash
   heroku config:set SECRET_KEY=your_secret_key
   heroku config:set PYTHONPATH=/app
   ```

## 👤 User Roles & Authentication

### Staff
- **Authentication**: Password only
- **Access**: Basic system access

### Security
- **Authentication**: Password + TOTP
- **Access**: Employee monitoring, visitor management

### Recruiter
- **Authentication**: Password + TOTP
- **Access**: Employee registration, visitor QR generation

### Admin
- **Authentication**: Password + Face Recognition + TOTP
- **Access**: Full system access, user management, role changes

## 🔧 Configuration

Key configuration options in `flask_info/constants.py`:

```python
# Face Recognition Settings
MODEL = 'hog'                    # 'hog' or 'cnn'
TOLERANCE = 0.6                  # Face matching tolerance

# Authentication Settings
max_attempts_google_auth = 3     # TOTP attempts
max_attempts_password = 5        # Password attempts

# QR Code Settings
time_interval = 43200            # QR code validity (seconds)
```

## 📱 Usage

### For Employees
1. **Login**: Use email and password
2. **Multi-factor**: Complete required authentication steps
3. **Generate QR**: Create entry QR codes as needed
4. **Check-in/out**: Use QR codes for building access

### For Administrators
1. **Employee Management**: Register new employees
2. **Face Registration**: Enroll admin users with face recognition
3. **Role Management**: Assign and modify user roles
4. **Monitoring**: View employee logs and present employees

### For Visitors
1. **QR Generation**: Admins/security can generate visitor QR codes
2. **Temporary Access**: Time-limited access with company identification

## 🔒 Security Considerations

- All passwords are hashed using Flask-User's secure hashing
- Face data is stored as encrypted pickle objects
- Session data is stored in PostgreSQL with encryption
- QR codes have time-based expiration
- Rate limiting prevents brute force attacks
- HTTPS enforcement on production

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👥 Team

- Arne Van Damme
- Arne Putzeys
- Ben Poelmans
- Dag Malstaf
- Marie-Johanna Schillemans
- Matisse Teuwen
- Tristan Toye

## 📞 Support

For support and questions, please contact the development team or create an issue in the repository.

---

**IDentiGate** - The key to security 🔑
