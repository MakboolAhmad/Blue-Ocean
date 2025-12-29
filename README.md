# Course Management System - NestJS Backend API

A comprehensive NestJS backend application for managing courses, categories, and subcategories with MongoDB/Mongoose, featuring full CRUD operations, relationship validation, pagination, filtering, and MongoDB aggregation pipelines.

## Features

 **Full CRUD Operations** for Category, SubCategory, and Course modules
 **Relationship Validation** - SubCategories must belong to Categories; Courses must link valid SubCategories to their Categories
 **Pagination, Sorting, Filtering, and Search** on all list endpoints
 **Soft Delete** using `isDeleted: boolean` field with restore functionality
 **MongoDB Transactions** for multi-document operations
 **MongoDB Aggregation Pipelines** for analytics and complex queries
 **DTO Validation** using `class-validator` and `class-transformer`
 **Global Exception Handling** with unified error responses
 **Proper HTTP Status Codes** (201 for creation, 200 for success, 404 for not found, etc.)
 **Well-organized Code Structure** following NestJS best practices
**Express.js Integration** via NestJS platform-express

## Project Structure

```
src/
├── common/
│   ├── dto/
│   │   └── pagination.dto.ts          # Pagination DTO
│   ├── filters/
│   │   └── all-exceptions.filter.ts   # Global exception handler
│   └── interfaces/
│       └── index.ts                   # Common interfaces
├── database/
│   ├── aggregation.service.ts         # MongoDB aggregation queries
│   └── analytics.controller.ts        # Analytics endpoints
├── modules/
│   ├── category/
│   │   ├── category.controller.ts
│   │   ├── category.service.ts
│   │   ├── category.module.ts
│   │   ├── dto/
│   │   │   └── category.dto.ts
│   │   └── schemas/
│   │       └── category.schema.ts
│   ├── subcategory/
│   │   ├── subcategory.controller.ts
│   │   ├── subcategory.service.ts
│   │   ├── subcategory.module.ts
│   │   ├── dto/
│   │   │   └── subcategory.dto.ts
│   │   └── schemas/
│   │       └── subcategory.schema.ts
│   └── course/
│       ├── course.controller.ts
│       ├── course.service.ts
│       ├── course.module.ts
│       ├── dto/
│       │   └── course.dto.ts
│       └── schemas/
│           └── course.schema.ts
├── app.module.ts                      # Root application module
└── main.ts                            # Application bootstrap
```

## Installation

### Prerequisites
- Node.js 20.x or higher
- MongoDB 4.4 or higher

### Setup Steps

1. **Install Dependencies**
   ```bash
   npm install
   ```

2. **Configure Environment Variables**
   ```bash
   # Edit .env file
   MONGODB_URI=mongodb://localhost:27017/course-management
   PORT=3000
   NODE_ENV=development
   ```

3. **Build the Project**
   ```bash
   npm run build
   ```

4. **Start the Server**
   - Development mode:
     ```bash
     npm run start:dev
     ```
   - Production mode:
     ```bash
     npm run build
     npm start
     ```

The API will be available at `http://localhost:3000/api/v1`

## API Endpoints

### Category Management
- `POST /api/v1/categories` - Create category
- `GET /api/v1/categories?page=1&limit=10&sort=-createdAt&search=tech` - List categories with pagination, sorting, search
- `GET /api/v1/categories/:id` - Get category by ID
- `PUT /api/v1/categories/:id` - Update category
- `DELETE /api/v1/categories/:id` - Soft delete category
- `DELETE /api/v1/categories/:id/permanent` - Permanent delete
- `PUT /api/v1/categories/:id/restore` - Restore soft deleted category

### SubCategory Management
- `POST /api/v1/subcategories` - Create subcategory (requires valid categoryId)
- `GET /api/v1/subcategories?page=1&limit=10&sort=-createdAt&search=web` - List subcategories with pagination
- `GET /api/v1/subcategories/category/:categoryId` - Get all subcategories for a category
- `GET /api/v1/subcategories/:id` - Get subcategory by ID
- `PUT /api/v1/subcategories/:id` - Update subcategory
- `DELETE /api/v1/subcategories/:id` - Soft delete
- `DELETE /api/v1/subcategories/:id/permanent` - Permanent delete
- `PUT /api/v1/subcategories/:id/restore` - Restore

### Course Management
- `POST /api/v1/courses` - Create course (validates subcategories belong to categories)
- `GET /api/v1/courses?page=1&limit=10&sort=-createdAt&search=node` - List courses with pagination
- `GET /api/v1/courses/category/:categoryId` - Get all courses for a category
- `GET /api/v1/courses/subcategory/:subCategoryId` - Get all courses for a subcategory
- `GET /api/v1/courses/:id` - Get course by ID
- `PUT /api/v1/courses/:id` - Update course (with validation)
- `DELETE /api/v1/courses/:id` - Soft delete
- `DELETE /api/v1/courses/:id/permanent` - Permanent delete
- `PUT /api/v1/courses/:id/restore` - Restore

### Analytics & Aggregation
- `GET /api/v1/analytics/category-subcategory-count` - Category with subcategory counts
- `GET /api/v1/analytics/category-subcategory-count-paginated?page=1&limit=10` - Paginated aggregation
- `GET /api/v1/analytics/category-subcategory-course-stats` - Advanced stats with course counts

## Request/Response Examples

### Create Category
**Request:**
```json
POST /api/v1/categories
{
  "name": "Programming",
  "description": "All programming related courses"
}
```

**Response:**
```json
{
  "success": true,
  "statusCode": 201,
  "message": "Category created successfully",
  "data": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "Programming",
    "description": "All programming related courses",
    "isDeleted": false,
    "createdAt": "2025-12-29T16:20:00.000Z",
    "updatedAt": "2025-12-29T16:20:00.000Z"
  }
}
```

### Create SubCategory
**Request:**
```json
POST /api/v1/subcategories
{
  "name": "Web Development",
  "description": "Front-end and back-end web development",
  "categoryId": "507f1f77bcf86cd799439011"
}
```

### Create Course (with Validation)
**Request:**
```json
POST /api/v1/courses
{
  "name": "Advanced Node.js",
  "description": "Master Node.js and build scalable applications",
  "instructor": "John Doe",
  "categoryIds": ["507f1f77bcf86cd799439011"],
  "subCategoryIds": ["507f1f77bcf86cd799439012"],
  "duration": 40,
  "price": 99.99
}
```

### List with Pagination & Filtering
**Request:**
```
GET /api/v1/courses?page=1&limit=10&sort=-price&search=node
```

**Response:**
```json
{
  "success": true,
  "statusCode": 200,
  "message": "Courses retrieved successfully",
  "data": {
    "data": [/* array of courses */],
    "total": 25,
    "page": 1,
    "limit": 10,
    "pages": 3
  }
}
```

## Key Features Explained

### Relationship Validation
- When creating a Course, all provided SubCategories are validated to ensure they belong to at least one of the selected Categories
- If validation fails, a 400 BadRequest error is returned

### Soft Delete
- Records are marked as `isDeleted: true` instead of being permanently removed
- List endpoints automatically filter out deleted records
- Deleted records can be restored using the `/restore` endpoint

### MongoDB Transactions
- When creating/updating Courses with multiple categories and subcategories, MongoDB transactions ensure data consistency
- If any validation fails, the entire operation is rolled back

### Aggregation Pipeline Examples

**Category with SubCategory Count:**
```javascript
db.categories.aggregate([
  { $match: { isDeleted: false } },
  {
    $lookup: {
      from: 'subcategories',
      localField: '_id',
      foreignField: 'categoryId',
      as: 'subCategories'
    }
  },
  {
    $addFields: {
      subCategoryCount: { $size: '$subCategories' }
    }
  },
  { $sort: { subCategoryCount: -1 } }
])
```

## Error Handling

All errors follow a consistent format:

```json
{
  "success": false,
  "statusCode": 400,
  "message": "Category with this name already exists",
  "error": "BadRequestException"
}
```

Common HTTP Status Codes:
- `200 OK` - Successful GET, PUT, DELETE
- `201 Created` - Successful POST
- `400 Bad Request` - Validation error or business logic error
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error

## Validation Rules

### Category
- `name`: Required, string, max 100 characters, unique
- `description`: Optional, string, max 500 characters

### SubCategory
- `name`: Required, string, max 100 characters, unique
- `categoryId`: Required, valid MongoDB ObjectId
- `description`: Optional, string, max 500 characters
- Must belong to an existing Category

### Course
- `name`: Required, string, max 100 characters, unique
- `categoryIds`: Required, array of valid MongoDB ObjectIds
- `subCategoryIds`: Required, array of valid MongoDB ObjectIds
- All SubCategories must belong to at least one selected Category
- `instructor`: Optional, string, max 100 characters
- `duration`: Optional, number >= 0
- `price`: Optional, number >= 0
- `description`: Optional, string, max 500 characters

## Query Parameters

### Pagination
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 10)

### Sorting
- `sort`: Field names with +/- prefix (-field for DESC, field for ASC)
- Example: `-createdAt,name` sorts by createdAt DESC, then name ASC

### Filtering & Search
- `search`: Text search on name field (case-insensitive regex)

## Technologies Used

- **NestJS** - Enterprise Node.js framework
- **MongoDB** - NoSQL database
- **Mongoose** - MongoDB object modeling
- **Express.js** - Web application framework (via NestJS)
- **TypeScript** - Static typing
- **class-validator** - DTO validation
- **class-transformer** - DTO transformation
- **Reflect Metadata** - Decorator support

## Architecture Patterns

### Factory Pattern
The service classes act as factories for creating domain objects and handling business logic. Each service encapsulates the creation and manipulation of its domain model.

### Module Pattern
NestJS modules organize related controllers, services, and entities into cohesive units. Each module can be independently tested and reused.

### MVC Pattern
- **Models**: Mongoose schemas (Category, SubCategory, Course)
- **Views**: API responses with unified format
- **Controllers**: Handle HTTP requests and delegate to services
- **Services**: Contain business logic and data operations

### Dependency Injection
All dependencies are injected through NestJS DI container, promoting loose coupling and testability.

## Future Enhancements

- User authentication and authorization
- Rate limiting
- Caching strategies
- Batch operations
- Advanced reporting and dashboards
- File uploads (course materials)
- Reviews and ratings
- Prerequisites management
- Course progress tracking

## License

ISC

## Support

For issues and questions, please refer to the NestJS documentation:
- [NestJS Documentation](https://docs.nestjs.com)
- [Mongoose Documentation](https://mongoosejs.com)
- [MongoDB Manual](https://docs.mongodb.com/manual)
