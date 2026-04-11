# Automated API Testing with Jest and Supertest

Automated integration tests for authentication API endpoints using Jest and Supertest with MongoDB.

## Project Overview

This project demonstrates modern testing practices for Node.js/Express APIs:
- **Integration Tests**: Full API endpoint testing with Supertest
- **Test Database**: Separate MongoDB test database with automatic cleanup
- **Lifecycle Management**: Proper setup and teardown using Jest hooks
- **Multiple Scenarios**: Both success and failure test cases
- **Server Refactoring**: Separated app configuration from server initialization

## Project Structure

```
.
├── app.js                 # Express app configuration (exported for testing)
├── server.js              # Server entry point (only runs in non-test env)
├── config/
│   └── database.js        # Database connection logic
├── models/
│   └── User.js            # User schema with password hashing
├── routes/
│   └── auth.js            # Authentication endpoints
├── tests/
│   └── auth.test.js       # Integration tests
├── .env                   # Environment variables
├── package.json           # Dependencies and test configuration
└── .gitignore
```

## Installation

### Prerequisites
- Node.js version 14+
- MongoDB running locally on port 27017 (or update MONGODB_URI in .env)

### Setup Steps

1. Clone the repository:
```bash
git clone https://github.com/yenibaraanoopjoel-sys/Automated-API-Testing-with-Jest-Supertest.git
cd Automated-API-Testing-with-Jest-Supertest
```

2. Install dependencies:
```bash
npm install
```

3. Configure environment variables (.env file is already created with defaults)

4. Ensure MongoDB is running:
```bash
mongod
```

## Running Tests

```bash
npm test
```

### Expected Output
```
Test Suites: 1 passed, 1 total
Tests:       11 passed, 11 total
Time:        ~3 seconds
```

## Test Coverage

### POST /api/auth/register
- ✅ Register with valid data (success)
- ✅ Register with existing email (failure - 409 Conflict)
- ✅ Register with missing fields (failure - 400 Bad Request)
- ✅ Register with mismatched passwords (failure - 400 Bad Request)
- ✅ Register with empty name (failure)

### POST /api/auth/login
- ✅ Login with correct credentials (success)
- ✅ Login with wrong password (failure - 401 Unauthorized)
- ✅ Login with non-existent email (failure - 401 Unauthorized)
- ✅ Login without email or password (failure - 400 Bad Request)
- ✅ Login without email (failure - 400 Bad Request)

### GET /health
- ✅ Health check endpoint verification

## Architecture Decisions

### 1. Separated app.js and server.js

**Why?**
- `app.js`: Exports Express application for testing (without calling listen())
- `server.js`: Only imports app and calls listen() for production

**Benefits:**
- Tests can import the Express app without starting a server
- Server startup logic is isolated from app configuration
- Better separation of concerns

### 2. Test Database Configuration

**Implementation:**
```javascript
const dbUri = process.env.NODE_ENV === 'test' 
  ? process.env.MONGODB_URI_TEST 
  : process.env.MONGODB_URI;
```

**Why?**
- Isolates tests from production data
- Prevents accidental data modification during testing
- Enables parallel test runs
- Easy to reset test data between tests

### 3. Lifecycle Hooks in Tests

```javascript
beforeAll(async () => {
  await connectDB();
});

afterEach(async () => {
  await User.deleteMany({});
});

afterAll(async () => {
  await disconnectDB();
});
```

**Why?**
- `beforeAll`: Establishes database connection once for all tests
- `afterEach`: Cleans up test data between tests (prevents test pollution)
- `afterAll`: Properly closes database connection

**Benefits:**
- **Isolation**: Each test starts with clean data
- **Performance**: Single database connection for all tests
- **Reliability**: No side effects between tests
- **Cleanup**: Resources properly released

### 4. Integration Testing with Supertest

Each test:
1. Makes actual HTTP requests to the API
2. Verifies status codes
3. Validates response structure
4. Checks success/error messages
5. Tests both happy path and error scenarios

## Key Testing Practices

### 1. Comprehensive Assertions
Each test validates multiple aspects:
```javascript
expect(response.status).toBe(201);
expect(response.body.success).toBe(true);
expect(response.body.token).toBeDefined();
expect(response.body.user.email).toBe('john@example.com');
```

### 2. Both Success and Failure Cases
- ✅ Happy path: Valid data
- ❌ Invalid input: Missing fields, wrong passwords
- ❌ Business logic: Duplicate emails, non-existent users

### 3. Test Data Isolation
```javascript
beforeEach(async () => {
  // Create user for login tests
});

afterEach(async () => {
  // Clear all test data
  await User.deleteMany({});
});
```

## Running the Server in Development

```bash
npm start
```

Server runs on `http://localhost:5000`

### Health Check
```bash
curl http://localhost:5000/health
```

## Environment Variables

```env
NODE_ENV=development
PORT=5000
MONGODB_URI=mongodb://localhost:27017/api_auth_db
MONGODB_URI_TEST=mongodb://localhost:27017/api_auth_test_db
JWT_SECRET=your_jwt_secret_key_change_in_production
```

## Dependencies

### Production
- **express**: Web framework
- **mongoose**: MongoDB ODM
- **bcryptjs**: Password hashing
- **jsonwebtoken**: JWT token creation
- **dotenv**: Environment variable management

### Development
- **jest**: Testing framework
- **supertest**: HTTP assertions

## Why Automated Testing Matters

### 1. **Regression Detection**
- Automatically catch bugs when code changes
- Prevents feature breakage

### 2. **Confidence in Deployments**
- Know that critical flows work before production
- Reduce manual testing time

### 3. **Documentation**
- Tests serve as usage examples
- New developers learn from test cases

### 4. **Faster Development**
- Catch errors immediately
- No need for manual Postman testing
- Quick feedback loop

### 5. **Scalability**
- Easy to add new test cases
- Handles complex scenarios
- Runs in seconds

## Next Steps (Potential Improvements)

- [ ] Add more complex scenarios (password reset, email verification)
- [ ] Implement authentication middleware tests
- [ ] Add rate limiting tests
- [ ] Set up CI/CD pipeline to auto-run tests
- [ ] Add test coverage reporting
- [ ] Implement database seeding for integration tests
- [ ] Add performance/load testing

## Troubleshooting

### MongoDB Connection Error
Ensure MongoDB is running:
```bash
mongod --version
```

### Module Not Found Errors
Reinstall dependencies:
```bash
rm -rf node_modules package-lock.json
npm install
```

### Tests Hanging
Check MongoDB connection and ensure the database is accessible.

## Contributing

1. Create a feature branch from `main`
2. Make your changes
3. Run tests: `npm test`
4. Commit with clear messages
5. Create a Pull Request

## License

ISC

## Author

- Created as part of automated API testing assignment
- Demonstrates Jest and Supertest integration testing practices
