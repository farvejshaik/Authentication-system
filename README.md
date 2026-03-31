# Authentication System

A comprehensive, secure authentication system built with Node.js, Express, and MongoDB. This system provides user registration, login, email verification, session management, and multi-device authentication capabilities.

## Features

- **User Registration & Login** with secure password hashing
- **Email Verification** using OTP (One-Time Password) system
- **JWT-based Authentication** with access and refresh tokens
- **Session Management** with device tracking
- **Multi-Device Support** - login from multiple devices
- **Secure Logout** - single device or all devices
- **Email Notifications** using Nodemailer with Gmail OAuth2
- **RESTful API** with proper error handling
- **Security Best Practices** including HTTP-only cookies, CORS, and input validation

## Tech Stack

### Backend

- **Node.js** - JavaScript runtime
- **Express.js** - Web framework
- **MongoDB** - NoSQL database with Mongoose ODM
- **JWT** - JSON Web Tokens for authentication
- **Nodemailer** - Email sending service
- **Crypto** - Built-in Node.js module for hashing
- **Morgan** - HTTP request logger
- **Cookie-parser** - Cookie parsing middleware
- **Dotenv** - Environment variable management

### Database

- **MongoDB Atlas** - Cloud-hosted MongoDB
- **Mongoose** - MongoDB object modeling

## Project Structure

```
Authentication/
├── src/
│   ├── config/
│   │   ├── config.js          # Environment configuration
│   │   └── database.js        # MongoDB connection
│   ├── controllers/
│   │   └── auth.controller.js # Authentication logic
│   ├── models/
│   │   ├── user.model.js      # User schema
│   │   ├── session.model.js   # Session schema
│   │   └── otp.model.js       # OTP schema
│   ├── routes/
│   │   └── auth.routes.js     # Authentication routes
│   ├── services/
│   │   └── email.service.js   # Email service
│   ├── utils/
│   │   └── utils.js           # Utility functions
│   └── app.js                 # Express app configuration
├── .env                       # Environment variables
├── .gitignore                 # Git ignore file
├── package.json               # Dependencies and scripts
├── server.js                  # Server entry point
└── README.md                  # This file
```

## Getting Started

### Prerequisites

- Node.js (v14 or higher)
- MongoDB Atlas account
- Gmail account with OAuth2 credentials

### Installation

1. **Clone the repository**

   ```bash
   git clone <repository-url>
   cd Authentication
   ```

2. **Install dependencies**

   ```bash
   npm install
   ```

3. **Set up environment variables**

   Create a `.env` file in the root directory with the following variables:

   ```env
   MONGO_URI=mongodb+srv://your-username:your-password@your-cluster.mongodb.net/your-database
   JWT_SECRET=your-super-secret-jwt-key
   GOOGLE_CLIENT_ID=your-google-client-id
   GOOGLE_CLIENT_SECRET=your-google-client-secret
   GOOGLE_REFRESH_TOKEN=your-google-refresh-token
   GOOGLE_USER=your-gmail-address
   ```

4. **Get Google OAuth2 Credentials**
   - Go to [Google Cloud Console](https://console.cloud.google.com/)
   - Create a new project or select an existing one
   - Enable Gmail API
   - Create OAuth2 credentials for a web application
   - Add your redirect URI (for testing, you can use `http://localhost`)
   - Generate a refresh token using OAuth2 playground

5. **Run the application**

   ```bash
   npm run dev
   ```

   The server will start on port 3000.

## API Documentation

### Base URL

```
http://localhost:3000/api/auth
```

### Authentication Endpoints

#### 1. Register User

```http
POST /api/auth/register
```

**Request Body:**

```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "securePassword123"
}
```

**Response (201):**

```json
{
  "message": "User registered successfully",
  "user": {
    "username": "john_doe",
    "email": "john@example.com",
    "verified": false
  }
}
```

#### 2. Login User

```http
POST /api/auth/login
```

**Request Body:**

```json
{
  "email": "john@example.com",
  "password": "securePassword123"
}
```

**Response (200):**

```json
{
  "message": "Logged in successfully",
  "user": {
    "username": "john_doe",
    "email": "john@example.com"
  },
  "accessToken": "access_token..."
}
```

**Cookies Set:**

- `refreshToken` (HTTP-only, secure, 7 days)

#### 3. Get Current User

```http
GET /api/auth/get-me
Authorization: Bearer <access_token>
```

**Response (200):**

```json
{
  "message": "User fetched successfully",
  "user": {
    "username": "john_doe",
    "email": "john@example.com",
    "verified": true
  }
}
```

#### 4. Refresh Access Token

```http
GET /api/auth/refresh-token
```

**Response (200):**

```json
{
  "message": "Token refreshed successfully",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

#### 5. Verify Email

```http
GET /api/auth/verify-email?email=john@example.com&otp=123456
```

**Response (200):**

```json
{
  "message": "Email verified successfully"
}
```

#### 6. Logout (Current Device)

```http
GET /api/auth/logout
```

**Response (200):**

```json
{
  "message": "Logged out successfully"
}
```

#### 7. Logout All Devices

```http
GET /api/auth/logout-all
```

**Response (200):**

```json
{
  "message": "Logged out from all devices successfully"
}
```

## Security Features

### Password Security

- SHA-256 hashing for password storage
- No plain text passwords stored in database

### Token Security

- JWT access tokens (15 minutes expiry)
- JWT refresh tokens (7 days expiry)
- HTTP-only cookies for refresh tokens
- Secure cookie flags (HTTPS only)
- SameSite strict policy

### Session Management

- Device tracking with IP address and User-Agent
- Session revocation capability
- Multi-device login support

### Email Security

- OTP-based email verification
- Gmail OAuth2 for secure email sending
- HTML email templates for better UX

### API Security

- Input validation and sanitization
- Proper error handling without information leakage
- CORS protection
- Request logging with Morgan

## Database Schema

### User Model

```javascript
{
  username: String (required, unique),
  email: String (required, unique),
  password: String (required, hashed),
  verified: Boolean (default: false),
  createdAt: Date,
  updatedAt: Date
}
```

### Session Model

```javascript
{
  user: ObjectId (ref: 'users'),
  refreshTokenHash: String (required),
  ip: String (required),
  userAgent: String (required),
  revoked: Boolean (default: false),
  createdAt: Date,
  updatedAt: Date
}
```

### OTP Model

```javascript
{
  email: String (required),
  user: ObjectId (ref: 'users'),
  otpHash: String (required),
  createdAt: Date,
  updatedAt: Date
}
```

## Authentication Flow

1. **Registration**
   - User provides username, email, password
   - System hashes password and creates user
   - Generates 6-digit OTP and sends via email
   - User remains unverified until OTP confirmation

2. **Email Verification**
   - User provides email and OTP
   - System validates OTP and marks user as verified
   - OTP record is automatically cleaned up

3. **Login**
   - User provides email and password
   - System validates credentials and verification status
   - Creates session with device tracking
   - Generates access and refresh tokens
   - Sets refresh token in HTTP-only cookie

4. **Token Refresh**
   - System validates refresh token from cookie
   - Checks if session is still active
   - Generates new access token

5. **Logout**
   - Single device: Revokes current session
   - All devices: Revokes all user sessions

## Development

### Scripts

```json
{
  "dev": "npx nodemon server.js"
}
```

### Environment Variables

The application requires the following environment variables:

- `MONGO_URI`: MongoDB connection string
- `JWT_SECRET`: Secret key for JWT signing
- `GOOGLE_CLIENT_ID`: Google OAuth2 client ID
- `GOOGLE_CLIENT_SECRET`: Google OAuth2 client secret
- `GOOGLE_REFRESH_TOKEN`: Google OAuth2 refresh token
- `GOOGLE_USER`: Gmail address for sending emails

### Error Handling

The system implements comprehensive error handling:

- Validation errors return 400 status
- Authentication errors return 401 status
- Conflict errors return 409 status
- Server errors return 500 status
- All errors include descriptive messages

## 📝 Future Enhancements

- [ ] Password reset functionality
- [ ] Two-factor authentication (2FA)
- [ ] Rate limiting for API endpoints
- [ ] Account lockout after failed attempts
- [ ] Social login integration (Google, GitHub, etc.)
- [ ] User profile management
- [ ] Role-based access control (RBAC)
- [ ] API documentation with Swagger/OpenAPI
- [ ] Unit and integration tests
- [ ] Docker containerization
- [ ] CI/CD pipeline setup

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the ISC License.

## Support

For any questions or issues, please open an issue on the GitHub repository.

---

Built with care by Farvej
