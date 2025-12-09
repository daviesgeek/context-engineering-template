## FEATURE:

Implement a comprehensive JWT-based authentication system for our Python FastAPI application.

### Core Requirements:
- User login endpoint (POST /auth/login) accepting email and password
- JWT access token generation with configurable expiration (default 24 hours)
- Refresh token support with longer expiration for seamless session extension
- Token refresh endpoint (POST /auth/refresh) to obtain new access tokens
- Logout endpoint (POST /auth/logout) to invalidate refresh tokens
- Protected route dependency for securing endpoints
- Password hashing using bcrypt for secure credential storage

### API Response Format:
- Successful login returns: `{ "access_token": "...", "refresh_token": "...", "token_type": "bearer" }`
- All auth errors return standard error format with appropriate HTTP status codes

## EXAMPLES:

- `examples/auth/oauth_flow.py` - Shows our existing OAuth2 implementation pattern. Follow the same service class structure and dependency injection approach.
- `examples/api/protected_routes.py` - Demonstrates how we protect routes with dependencies. Use this pattern for the JWT auth dependency.
- `examples/api/responses.py` - Standard API response formatting. All endpoints should use these response helpers.

## DOCUMENTATION:

- https://jwt.io/introduction/ - JWT fundamentals and best practices
- https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/ - FastAPI JWT implementation guide
- https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html - Password storage best practices

## OTHER CONSIDERATIONS:

### Integration Requirements:
- Must integrate with existing User model in `src/models/user.py` (has email, password_hash fields)
- Store refresh tokens in Redis using existing client at `src/db/redis.py`
- Follow API response patterns from `src/api/responses.py`

### Security Considerations:
- JWT secret key should come from environment variable `JWT_SECRET_KEY`
- Implement token blacklisting for logout functionality
- Consider rate limiting on auth endpoints (document but don't implement)

### Testing:
- Include unit tests for all auth service methods
- Include integration tests for auth endpoints
- Test both happy path and error scenarios

### Out of Scope (do not implement):
- User registration (separate feature)
- Password reset flow
- OAuth2 social providers
- Multi-factor authentication
