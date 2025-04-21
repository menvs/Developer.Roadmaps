# Tips and Techniques to Improve Query Performance

Performance is critical in real-world applications. Here are some techniques to optimize your EF Core queries:

## `AsNoTracking()` and Tracking Behavior

- By default, EF Core tracks changes to the entities it retrieves. This allows for automatic change detection and updates when `SaveChanges()` is called.
- **`AsNoTracking()`:** Disables change tracking for a query. Use this when you are only reading data and don't intend to modify the retrieved entities. This can significantly improve performance and reduce memory consumption, especially for large datasets.

```csharp
// Tracking query
var trackedBlogs = context.Blogs.ToList();

// No-tracking query
var readOnlyBlogs = context.Blogs.AsNoTracking().ToList();
```

**Best Practice:** Use `AsNoTracking()` for read-only operations to improve performance. Only track entities when you intend to update them.

## Using Projections with `.Select()`

- The `.Select()` method allows you to shape the data you retrieve from the database, selecting only the properties you need.
- This reduces the amount of data transferred and can significantly improve query performance.

```csharp
// Fetching only the blog URL and title
var blogUrlsAndTitles = context.Blogs
    .Select(b => new { b.Url, b.Title })
    .ToList();

// Projecting to a DTO
var blogDtos = context.Blogs
    .Select(b => new BlogDto { Id = b.BlogId, Name = b.Title })
    .ToList();
```

**Best Practice:** Always project to anonymous types or DTOs when you only need a subset of the entity properties.

## Batch Queries

- EF Core automatically groups some related operations into a single database round trip (batching).
- For explicit batching of updates or deletes, consider using libraries like `EFCore.BulkExtensions`.

## Avoiding N+1 Issues with `.Include()`

- The N+1 problem occurs when you fetch a list of entities and then, for each entity, you execute a separate query to load related data.
- **`.Include()`:** Performs an eager load of related entities in a single query (using a JOIN in SQL), avoiding the N+1 problem.

```csharp
// Without Include, this might result in N+1 queries
var blogsWithPosts = context.Blogs.ToList();
foreach (var blog in blogsWithPosts)
{
    Console.WriteLine($"Blog: {blog.Title}, Post Count: {blog.Posts.Count}"); // Triggers a separate query for each blog's posts
}

// Using Include to load posts in a single query
var blogsWithPostsEagerLoaded = context.Blogs.Include(b => b.Posts).ToList();
foreach (var blog in blogsWithPostsEagerLoaded)
{
    Console.WriteLine($"Blog: {blog.Title}, Post Count: {blog.Posts.Count}"); // Posts are already loaded
}
```

- **`.ThenInclude()`:** Used to eagerly load deeper levels of related entities.

```csharp
var blogsWithPostsAndAuthors = context.Blogs
    .Include(b => b.Posts)
        .ThenInclude(p => p.Author)
    .ToList();
```

- **`.Load()` and Explicit Loading:** For scenarios where you don't always need related data, you can use `.Load()` to explicitly load related entities for a specific entity instance after the initial query.

```csharp
var blog = context.Blogs.Find(1);
context.Entry(blog).Collection(b => b.Posts).Load();
```

- **Split Queries:** For complex includes, EF Core might generate very large SQL queries. You can use `AsSplitQuery()` to force EF Core to generate separate SQL queries for the main entity and its included collections, which can sometimes improve performance.

```csharp
var blogsWithPostsSplit = context.Blogs.Include(b => b.Posts).AsSplitQuery().ToList();
```

**Best Practice:** Use `.Include()` to eagerly load frequently accessed related data to avoid the N+1 problem. Consider `.ThenInclude()` for deeper relationships. Evaluate `AsSplitQuery()` for complex include scenarios.

## Filtering at the Database Level

- Perform filtering using `.Where()` in your LINQ queries. EF Core will translate this to a `WHERE` clause in the SQL query, ensuring that only the necessary data is retrieved from the database.
- Avoid filtering in memory after retrieving all the data.

```csharp
// Filtering at the database level (efficient)
var activeBlogs = context.Blogs.Where(b => b.IsActive).ToList();

// Filtering in memory (less efficient if Blogs table is large)
var allBlogs = context.Blogs.ToList();
var activeBlogsInMemory = allBlogs.Where(b => b.IsActive).ToList();
```

**Best Practice:** Always apply filtering (`.Where()`) as early as possible in your query to reduce the amount of data processed by the database and transferred to your application.

## Raw SQL and Stored Procedures When Needed

- EF Core allows you to execute raw SQL queries using `FromSqlRaw()`. This can be useful for complex queries or when you need to leverage database-specific features.

```csharp
var blogs = context.Blogs.FromSqlRaw("SELECT * FROM Blogs WHERE Title LIKE '%example%'").ToList();

// With parameters to prevent SQL injection
var searchTerm = "%example%";
var blogsWithParam = context.Blogs.FromSqlRaw("SELECT * FROM Blogs WHERE Title LIKE {0}", searchTerm).ToList();
```

- You can also call stored procedures using `FromSqlRaw()` or by mapping them in your `DbContext`.

```csharp
// Executing a stored procedure
var blogsFromProc = context.Blogs.FromSqlRaw("EXECUTE GetBlogsByCreationDate {0}", DateTime.Now.AddDays(-30)).ToList();
```

**Caution:** Use raw SQL and stored procedures judiciously, as they can make your code less portable and harder to maintain if not managed carefully.

## Indexing and Query Profiling

- **Indexing:** Ensure that your database tables have appropriate indexes on the columns frequently used in `WHERE` clauses, `ORDER BY` clauses, and join conditions. Proper indexing is crucial for database performance.
- **Query Profiling:** Use database-specific tools (e.g., SQL Server Profiler, Azure Data Studio's query analyzer, PostgreSQL's `EXPLAIN`) to analyze the SQL queries generated by EF Core and identify performance bottlenecks. EF Core also provides logging capabilities that can help you see the generated SQL.

```csharp
// Enabling logging in DbContextOptionsBuilder
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseSqlServer("YourConnectionString")
        .LogTo(Console.WriteLine, (parameters) => parameters.CommandText.Contains("FROM"))
        .EnableSensitiveDataLogging(); // Only for development!
}
```

**Best Practice:** Analyze your database query performance using profiling tools and ensure that your database tables are properly indexed.

## Common Pitfalls to Avoid with EF Core

- **Lazy Loading Surprises:** While EF Core supports lazy loading, relying on it heavily can lead to the N+1 problem if you're not careful about how you access navigation properties. It's generally better to be explicit with eager loading or projections.
- **Unnecessary Tracking:** Loading entities with the default tracking behavior when you only need to read data can impact performance and memory usage. Use `AsNoTracking()` when appropriate.
- **Inefficient Queries:** Writing complex LINQ queries that translate into inefficient SQL can cause performance issues. Always review the generated SQL.
- **Global AsNoTracking:** Applying `.AsNoTracking()` globally