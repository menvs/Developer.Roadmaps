# Best Practices for Using EF Core

Here are some essential best practices to follow when using EF Core in your projects:

### Organizing `DbContext` and Models

- **Single Responsibility**: Create separate DbContext classes for different bounded contexts (e.g., OrderContext, ProductContext).
- **Folder Structure**: Organize models in a Models or Entities folder, with subfolders for domains (e.g., Models/Orders, Models/Products).
- **Configuration:** Use the `OnModelCreating` method in your `DbContext` or separate `IEntityTypeConfiguration<TEntity>` classes to configure entity mappings (table names, column properties, relationships, etc.) for better organization.

```csharp
public class BlogEntityTypeConfiguration : IEntityTypeConfiguration<Blog>
{
    public void Configure(EntityTypeBuilder<Blog> builder)
    {
        builder.ToTable("T_Blogs");
        builder.HasKey(b => b.BlogId);
        builder.Property(b => b.Url).IsRequired().HasMaxLength(200);
        // ... other configurations
    }
}

// In your DbContext's OnModelCreating:
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfiguration(new BlogEntityTypeConfiguration());
    modelBuilder.ApplyConfiguration(new PostEntityTypeConfiguration());
}
```

### DTOs vs Entities

- **Entities:** Represent your database tables and have a lifecycle managed by the `DbContext`. They might contain navigation properties representing relationships.
- **DTOs (Data Transfer Objects):** Simple objects used to transfer data between layers of your application (e.g., controllers and services) or to the client. They are often flattened versions of your entities and can be tailored to specific use cases.

**Best Practice:** Use DTOs for data transfer to avoid exposing your entire entity graph and to shape the data according to the needs of the consumer. Map between entities and DTOs using libraries like AutoMapper.

### Separation of Concerns with Repository or Service Patterns

- **Repository Pattern:** Encapsulates the logic for accessing data from the data source (in this case, EF Core). It provides an abstraction layer over the `DbContext`, making your application code more testable and less dependent on the specific ORM.
- **Service Layer:** Contains the business logic of your application and orchestrates interactions between repositories and other services.

```csharp
public interface IBlogRepository
{
    Task<IEnumerable<Blog>> GetAllBlogsAsync();
    Task<Blog> GetBlogByIdAsync(int id);
    void AddBlog(Blog blog);
    void UpdateBlog(Blog blog);
    void DeleteBlog(int id);
}

public class BlogRepository : IBlogRepository
{
    private readonly BloggingContext _context;

    public BlogRepository(BloggingContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Blog>> GetAllBlogsAsync()
    {
        return await _context.Blogs.ToListAsync();
    }

    // ... other methods
}

public class BlogService
{
    private readonly IBlogRepository _blogRepository;

    public BlogService(IBlogRepository blogRepository)
    {
        _blogRepository = blogRepository;
    }

    public async Task<IEnumerable<BlogDto>> GetActiveBlogsAsync()
    {
        var blogs = await _blogRepository.GetAllBlogsAsync();
        return blogs.Where(b => b.IsActive).Select(b => new BlogDto { /* ... */ });
    }
}
```

**Best Practice:** Implement the Repository pattern (or a similar data access abstraction) and a Service Layer to promote separation of concerns, improve testability, and make your application more maintainable.

### Avoiding Over-fetching / Under-fetching Data

- **Over-fetching:** Retrieving more data than you actually need, leading to unnecessary database load and slower performance. For example, loading all related posts when you only need the blog title.
- **Under-fetching:** Not retrieving enough data in the initial query, requiring additional database queries to fetch related information (the N+1 problem).

**Best Practice:** Be mindful of the data you are loading. Use projections (`.Select()`) to retrieve only the necessary columns. Use eager loading (`.Include()`) or explicit/lazy loading (with caution) to fetch related entities when needed, avoiding the N+1 problem.

```csharp
var productDtos = await context.Products
    .Select(p => new ProductDto { Id = p.Id, Name = p.Name, Price = p.Price })
    .ToListAsync();
```