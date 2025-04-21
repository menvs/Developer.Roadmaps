# Relationships & Navigation Properties

## **Relationships in Databases**

In relational databases, tables are often linked together through relationships. Common relationships

- **One-to-One:** One record in table A is related to at most one record in table B, and vice versa. Example: A person can have at most one detailed profile, and a detailed profile belongs to only one person.
- **One-to-Many:** One record in table A can be related to many records in table B, but one record in table B is related to only one record in table A. Example: An author can write many posts, but a post belongs to only one author.
- **Many-to-Many:** Many records in table A can be related to many records in table B, and vice versa. This relationship is often implemented through a junction table. Example: Many students can enroll in many courses, and a course can have many students.

## **Navigation Properties in EF Core**

In EF Core, navigation properties are reference properties in your entity classes that allow you to navigate between related entities. They don't map directly to database columns but represent the relationships that have been defined.

There are two main types of navigation properties:

- **Reference Navigation Property:** Represents a relationship where the current entity is related to a *single* other entity. Commonly used for One-to-One relationships and the "one" side of One-to-Many relationships. For example, in a `Post` class, you might have an `Author` property of type `Blog`.
- **Collection Navigation Property:** Represents a relationship where the current entity is related to *multiple* other entities. Commonly used for the "many" side of One-to-Many relationships and both sides of Many-to-Many relationships. For example, in a `Blog` class, you might have a `Posts` property of type `ICollection<Post>`.

## **Defining Relationships and Navigation Properties**

You can define relationships and navigation properties in EF Core using one of two main ways:

### **Convention-based Configuration:**

EF Core tries to automatically discover and configure relationships based on naming conventions in your entity classes. For example:

- If you have a foreign key property named according to convention (e.g., `BlogId` in the `Post` class) and a reference navigation property with the same name as the related class (`Blog` in the `Post` class), EF Core will automatically set up a One-to-Many relationship between `Blog` and `Post`.
- Similarly, a collection navigation property with the pluralized name of the related class (`Posts` in the `Blog` class) will be understood by EF Core as the "many" side of the relationship.

```csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
    public ICollection<Post> Posts { get; set; } // Collection navigation property
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    public int BlogId { get; set; } // Foreign key property
    public Blog Blog { get; set; } // Reference navigation property
}
```

### **Fluent API Configuration:**

You can use the Fluent API in the `OnModelCreating` method of your `DbContext` to configure relationships explicitly and with more customization. This allows you to control foreign key names, constraints, delete behavior, and many other aspects of the relationship.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Post>()
        .HasOne(p => p.Blog) // Define the "one" side of the relationship
        .WithMany(b => b.Posts) // Define the "many" side of the relationship
        .HasForeignKey(p => p.BlogId) // Specify the foreign key
        .OnDelete(DeleteBehavior.Cascade); // Configure delete behavior

    // Configure a One-to-One relationship (e.g., between Person and PersonDetail)
    modelBuilder.Entity<Person>()
        .HasOne(p => p.PersonDetail)
        .WithOne(pd => pd.Person)
        .HasForeignKey<PersonDetail>(pd => pd.PersonId);

    // Configure a Many-to-Many relationship (e.g., between Student and Course)
    modelBuilder.Entity<StudentCourse>() // Junction table (if you create it manually)
        .HasKey(sc => new { sc.StudentId, sc.CourseId });
    modelBuilder.Entity<StudentCourse>()
        .HasOne<Student>(sc => sc.Student)
        .WithMany(s => s.StudentCourses)
        .HasForeignKey(sc => sc.StudentId);
    modelBuilder.Entity<StudentCourse>()
        .HasOne<Course>(sc => sc.Course)
        .WithMany(c => c.StudentCourses)
        .HasForeignKey(sc => sc.CourseId);

    // If you use collection navigation properties directly in Student and Course:
    modelBuilder.Entity<Student>()
        .HasMany(s => s.Courses)
        .WithMany(c => c.Students)
        .UsingEntity(j => j.ToTable("StudentCourse")); // EF Core will automatically create the junction table
}

public class Person
{
    public int PersonId { get; set; }
    public string Name { get; set; }
    public PersonDetail PersonDetail { get; set; } // Reference navigation property
}

public class PersonDetail
{
    public int PersonDetailId { get; set; }
    public string Address { get; set; }
    public int PersonId { get; set; } // Foreign key
    public Person Person { get; set; } // Reference navigation property
}

public class Student
{
    public int StudentId { get; set; }
    public string StudentName { get; set; }
    public ICollection<StudentCourse> StudentCourses { get; set; } // Collection navigation property (for the junction table)
    public ICollection<Course> Courses { get; set; } // Collection navigation property (for direct Many-to-Many)
}

public class Course
{
    public int CourseId { get; set; }
    public string CourseName { get; set; }
    public ICollection
```

## Best Practices for Relationships and Navigation Properties

- **Use Fluent API for Complex Relationships**: Conventions and Data Annotations are limited; Fluent API offers full control.
- **Specify Delete Behavior**: Use OnDelete to define cascade behavior (e.g., DeleteBehavior.Cascade, DeleteBehavior.Restrict).
- **Avoid Overusing .Include()**: Deep or excessive eager loading can result in large, slow queries. Use projections or split queries instead.
- **Minimize Lazy Loading**: Prefer eager or explicit loading to avoid N+1 issues.
- **Use Projections for Read-Only Queries**: Select only needed data to reduce memory and query overhead.

```csharp
var customerDtos = await context.Customers
    .Select(c => new
    {
        c.Id,
        c.Name,
        OrderCount = c.Orders.Count
    })
    .ToListAsync();
```

- **Define Indexes on Foreign Keys**: Improve query performance for joins and filters.

```csharp
modelBuilder.Entity<Order>()
    .HasIndex(o => o.CustomerId);
```

- **Use Owned Types for One-to-One**: For tightly coupled one-to-one relationships, consider owned types to simplify configuration.

```csharp
public class User
{
    public int Id { get; set; }
    public Address Address { get; set; } *// Owned type*
}

modelBuilder.Entity<User>()
    .OwnsOne(u => u.Address);
```

---

## References

- [Microsoft Docs: Relationships in EF Core](https://learn.microsoft.com/en-us/ef/core/modeling/relationships)
- [Microsoft Docs: Navigation Properties](https://learn.microsoft.com/en-us/ef/core/modeling/relationships/navigation-properties)
- [EF Core Loading Related Data](https://learn.microsoft.com/en-us/ef/core/querying/related-data)
- [MiniProfiler for .NET](https://miniprofiler.com/dotnet/)