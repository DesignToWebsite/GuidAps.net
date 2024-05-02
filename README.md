# GuidAps.net
In ASP.NET, you can establish a relationship between two tables (or entities) using primary keys and foreign keys. Here is a step-by-step guide to create a relationship between tables using ASP.NET Core and Entity Framework Core:

### Step 1: Define the Entities

1. **Create the `Student` Entity**:
    
    ```csharp
    using System.Collections.Generic;
    using System.ComponentModel.DataAnnotations;

    namespace INNOMIATE_API.Models;

    public class Student
    {
        [Key]
        public int StudentId { get; set; }  // Primary key
        
        public required string Name { get; set; }
        public int Age { get; set; }
        
        // Navigation property for the many-to-many relationship with Courses
        public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
    }
    ```

2. **Create the `Course` Entity**:

    ```csharp
    using System.Collections.Generic;
    using System.ComponentModel.DataAnnotations;

    namespace INNOMIATE_API.Models;

    public class Course
    {
        [Key]
        public int CourseId { get; set; }  // Primary key
        
        public required string CourseName { get; set; }
        
        // Navigation property for the many-to-many relationship with Students
        public virtual ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
    }
    ```

3. **Create the `Enrollment` Entity** (the join table):

    ```csharp
    using System.ComponentModel.DataAnnotations;
    using System.ComponentModel.DataAnnotations.Schema;

    namespace INNOMIATE_API.Models;

    public class Enrollment
    {
        [Key]
        public int EnrollmentId { get; set; } // Primary key for the join table
        
        // Foreign key referencing Student
        [ForeignKey("Student")]
        public int StudentId { get; set; }
        public virtual Student Student { get; set; }  // Navigation property
        
        // Foreign key referencing Course
        [ForeignKey("Course")]
        public int CourseId { get; set; }
        public virtual Course Course { get; set; }  // Navigation property
    }
    ```

### Step 2: Update the Database Context

1. **Add DbSet properties to the `ApplicationDbContext`**:

    ```csharp
    using INNOMIATE_API.Models;
    using Microsoft.EntityFrameworkCore;

    namespace INNOMIATE_API.Data;

    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
        {
        }

        public DbSet<Student> Students { get; set; }
        public DbSet<Course> Courses { get; set; }
        public DbSet<Enrollment> Enrollments { get; set; }
    }
    ```

### Step 3: Perform Database Migrations

1. **Add a migration**:

    - Open a terminal or command prompt, navigate to your project directory, and run the following command to create a migration:

        ```shell
        dotnet ef migrations add InitialCreate
        ```

2. **Update the database**:

    - Once the migration is created, apply the migration to your database:

        ```shell
        dotnet ef database update
        ```

### Step 4: Use the Relationships in Your Code

You can now use the relationships in your code:

- **Query data**: You can query related data using the navigation properties.
    ```csharp
    // Get student enrollments
    var studentEnrollments = await _context.Students
        .Include(s => s.Enrollments)
            .ThenInclude(e => e.Course)
        .FirstOrDefaultAsync(s => s.StudentId == studentId);
    ```

- **Add data**: You can add new enrollments by creating a new `Enrollment` entity and saving it to the context.

    ```csharp
    var enrollment = new Enrollment
    {
        StudentId = studentId,
        CourseId = courseId
    };
    _context.Enrollments.Add(enrollment);
    await _context.SaveChangesAsync();
    ```

### Conclusion

These are the basic steps to set up a relationship using primary and foreign keys in ASP.NET Core and Entity Framework Core. Define your entities with the appropriate primary keys and foreign keys, update your `ApplicationDbContext`, perform database migrations, and then you can use the relationships in your code for querying and adding data.
