# Cursor Rules for API Development with Next.js

You are an expert Next.js API developer. You will follow these rules when writing API code with this tech stack.

## Core Technologies
- **TypeScript** - For all code
- **Next.js** (App Router) - For server-side API route handlers
- **Prisma** - For database operations
- **PostgreSQL** - As the database
- **NextAuth.js / Auth.js** - For authentication

## API Structure and Organization

### Route Organization
- Place API routes in `app/api` directory
- Use folder structure for route segmentation (e.g., `app/api/users/[id]/route.ts`)
- Implement HTTP methods as named exports (`GET`, `POST`, `PUT`, `DELETE`, etc.)
- Use dynamic segments with square brackets for parameters (e.g., `[id]`, `[slug]`)

### Request/Response Handling
- Use the Edge Runtime when possible for faster global response times
- Utilize `NextResponse` for standardized responses
- Implement consistent error handling with proper status codes
- Validate request data before processing

### Edge vs. Node.js Runtime
- Use Edge Runtime for simple, stateless operations
- Use Node.js Runtime for complex operations requiring additional NPM modules
- Specify runtime with `export const runtime = 'edge'` or `'nodejs'`

## Data and Authentication

### Database Operations
- Use Prisma Client for all database interactions
- Create separate service files for database logic
- Implement transactions for multi-step operations
- Handle database errors gracefully

### Authentication and Authorization
- Use NextAuth.js middleware for protected routes
- Implement role-based access control where needed
- Verify session status at the beginning of each route handler
- Keep sensitive operations server-side only

## API Design and Best Practices

### API Response Format
- Maintain consistent response structure:
  ```typescript
  {
    success: boolean;
    data?: any;
    error?: {
      code: string;
      message: string;
    }
  }
  ```
- Use proper HTTP status codes (200, 201, 400, 401, 403, 404, 500)
- Include pagination metadata for list endpoints

### Performance and Optimization
- Implement caching strategies using Next.js cache directives
- Optimize database queries (select only needed fields)
- Use connection pooling for database operations
- Handle rate limiting for public-facing APIs

### Error Handling
- Create a centralized error handling utility
- Categorize errors (validation, authentication, database, etc.)
- Log errors with appropriate detail for debugging
- Return user-friendly error messages in responses

### Security Considerations
- Sanitize all input data before processing
- Use parameterized queries to prevent SQL injection
- Implement CORS policies properly
- Never expose sensitive information in responses

## Code Style and TypeScript Usage

### TypeScript Patterns
- Define interfaces for request/response data
- Use zod or similar for runtime validation
- Create utility types for common patterns
- Export shared types to dedicated files

### Naming Conventions
- Use **kebab-case** for API route directories
- Use **camelCase** for variables and functions
- Prefix async functions with descriptive verbs (e.g., `fetchUser`, `createPost`)
- Name route handler files as `route.ts`

### Code Structure
- Separate business logic from route handlers
- Implement early returns for validation failures
- Use try/catch blocks consistently
- Document complex logic with comments

## Example

```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { prisma } from '@/lib/prisma';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

// Specify runtime - use Edge for better performance
export const runtime = 'edge';

// Define validation schema
const userUpdateSchema = z.object({
  name: z.string().min(2).optional(),
  email: z.string().email().optional(),
  role: z.enum(['USER', 'ADMIN']).optional(),
});

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    // Verify authentication
    const session = await getServerSession(authOptions);
    if (!session) {
      return NextResponse.json(
        { 
          success: false, 
          error: { 
            code: 'UNAUTHORIZED', 
            message: 'Authentication required' 
          } 
        },
        { status: 401 }
      );
    }

    // Fetch user from database
    const user = await prisma.user.findUnique({
      where: { id: params.id },
      select: {
        id: true,
        name: true,
        email: true,
        role: true,
        createdAt: true,
        // Exclude sensitive fields
      },
    });

    if (!user) {
      return NextResponse.json(
        { 
          success: false, 
          error: { 
            code: 'NOT_FOUND', 
            message: 'User not found' 
          } 
        },
        { status: 404 }
      );
    }

    // Return successful response
    return NextResponse.json({ success: true, data: user });
  } catch (error) {
    console.error('Error fetching user:', error);
    return NextResponse.json(
      { 
        success: false, 
        error: { 
          code: 'SERVER_ERROR', 
          message: 'Failed to fetch user data' 
        } 
      },
      { status: 500 }
    );
  }
}

export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    // Verify authentication
    const session = await getServerSession(authOptions);
    if (!session) {
      return NextResponse.json(
        { 
          success: false, 
          error: { 
            code: 'UNAUTHORIZED', 
            message: 'Authentication required' 
          } 
        },
        { status: 401 }
      );
    }

    // Check authorization (admin or self)
    if (session.user.role !== 'ADMIN' && session.user.id !== params.id) {
      return NextResponse.json(
        { 
          success: false, 
          error: { 
            code: 'FORBIDDEN', 
            message: 'You do not have permission to update this user' 
          } 
        },
        { status: 403 }
      );
    }

    // Parse and validate request body
    const body = await request.json();
    const validationResult = userUpdateSchema.safeParse(body);
    
    if (!validationResult.success) {
      return NextResponse.json(
        { 
          success: false, 
          error: { 
            code: 'VALIDATION_ERROR', 
            message: 'Invalid request data',
            details: validationResult.error.errors
          } 
        },
        { status: 400 }
      );
    }

    // Update user in database
    const updatedUser = await prisma.user.update({
      where: { id: params.id },
      data: validationResult.data,
      select: {
        id: true,
        name: true,
        email: true,
        role: true,
        updatedAt: true,
      },
    });

    // Return successful response
    return NextResponse.json({ 
      success: true, 
      data: updatedUser 
    });
  } catch (error) {
    // Handle specific error types
    if (error.code === 'P2025') {
      return NextResponse.json(
        { 
          success: false, 
          error: { 
            code: 'NOT_FOUND', 
            message: 'User not found' 
          } 
        },
        { status: 404 }
      );
    }

    console.error('Error updating user:', error);
    return NextResponse.json(
      { 
        success: false, 
        error: { 
          code: 'SERVER_ERROR', 
          message: 'Failed to update user data' 
        } 
      },
      { status: 500 }
    );
  }
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    // Verify authentication and admin role
    const session = await getServerSession(authOptions);
    if (!session) {
      return NextResponse.json(
        { 
          success: false, 
          error: { 
            code: 'UNAUTHORIZED', 
            message: 'Authentication required' 
          } 
        },
        { status: 401 }
      );
    }

    // Only admins can delete users
    if (session.user.role !== 'ADMIN') {
      return NextResponse.json(
        { 
          success: false, 
          error: { 
            code: 'FORBIDDEN', 
            message: 'Admin privileges required' 
          } 
        },
        { status: 403 }
      );
    }

    // Delete user from database
    await prisma.user.delete({
      where: { id: params.id },
    });

    // Return successful response
    return NextResponse.json(
      { success: true, data: { message: 'User deleted successfully' } },
      { status: 200 }
    );
  } catch (error) {
    // Handle specific error types
    if (error.code === 'P2025') {
      return NextResponse.json(
        { 
          success: false, 
          error: { 
            code: 'NOT_FOUND', 
            message: 'User not found' 
          } 
        },
        { status: 404 }
      );
    }

    console.error('Error deleting user:', error);
    return NextResponse.json(
      { 
        success: false, 
        error: { 
          code: 'SERVER_ERROR', 
          message: 'Failed to delete user' 
        } 
      },
      { status: 500 }
    );
  }
}
```
