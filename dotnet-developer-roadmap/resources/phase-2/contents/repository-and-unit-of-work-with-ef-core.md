# Repository & Unit of Work with EF Core

# **Repository Pattern**

**Definition:** The Repository Pattern is a class or a set of classes that acts as an intermediary between the business logic layer of the application and the data access layer (in this case, EF Core). The repository is responsible for performing CRUD (Create, Read, Update, Delete) operations on entities.

**Goals:**

- **Separation of Concerns:** Separates business logic from data access logic. The business layer doesn't need to know how the data is stored or retrieved.
- **Abstraction of the Data Layer:** Provides a simple and consistent interface for the business layer to work with data. This helps hide the complexity of EF Core and the database.
- **Increased Testability:** You can easily create mock repositories to test the business logic without interacting with the actual database.
- **Ease of Data Layer Changes:** If you decide to change the ORM (e.g., from EF Core to Dapper) or change the database, you only need to modify the repository implementations without affecting the business layer.
- **Reusability of Data Access Logic:** Common data access operations can be implemented once in the repository and reused in many parts of the application.

## **Basic Implementation with EF Core:**

1. **Define the Repository Interface:**
    
    ```csharp
    public interface IBlogRepository
    {
        Task<Blog> GetByIdAsync(int id);
        Task<IEnumerable<Blog>> GetAllAsync();
        Task AddAsync(Blog entity);
        Task UpdateAsync(Blog entity);
        Task DeleteAsync(int id);
    }
    ```
    
2. **Implement a Concrete Repository (using EF Core):**C#
    
    ```csharp
    public class BlogRepository : IBlogRepository
    {
        private readonly BloggingContext _dbContext;
    
        public BlogRepository(BloggingContext dbContext)
        {
            _dbContext = dbContext;
        }
    
        public async Task<Blog> GetByIdAsync(int id)
        {
            return await _dbContext.Blogs.FindAsync(id);
        }
    
        public async Task<IEnumerable<Blog>> GetAllAsync()
        {
            return await _dbContext.Blogs.ToListAsync();
        }
    
        public async Task AddAsync(Blog entity)
        {
            await _dbContext.Blogs.AddAsync(entity);
        }
    
        public async Task UpdateAsync(Blog entity)
        {
            _dbContext.Entry(entity).State = EntityState.Modified;
        }
    
        public async Task DeleteAsync(int id)
        {
            var blog = await _dbContext.Blogs.FindAsync(id);
            if (blog != null)
            {
                _dbContext.Blogs.Remove(blog);
            }
        }
    }
    ```
    
3. **Use the Repository in a Service or Controller:**
    
    ```csharp
    public class BlogService
    {
        private readonly IBlogRepository _blogRepository;
    
        public BlogService(IBlogRepository blogRepository)
        {
            _blogRepository = blogRepository;
        }
    
        public async Task<BlogDto> GetBlogDtoByIdAsync(int id)
        {
            var blog = await _blogRepository.GetByIdAsync(id);
            if (blog == null)
            {
                return null;
            }
            return new BlogDto { Id = blog.BlogId, Title = blog.Title, Url = blog.Url };
        }
    
        // ... other business logic methods
    }
    ```
    

---

# **Unit of Work Pattern**

**Definition:** The Unit of Work Pattern manages one or more operations that are eventually saved as a single unit of work (e.g., a transaction). It tracks all the changes made to entities within a "unit" and only saves them to the database when that unit is complete.

**Goals:**

- **Ensuring Data Consistency:** Groups multiple related data operations into a single transaction. If any operation fails, the entire unit of work is rolled back, ensuring data integrity.
- **Reducing `SaveChanges()` Calls:** Instead of calling `SaveChanges()` after each repository operation, you call it once at the end of the unit of work. This can improve performance by reducing the number of database interactions.
- **Managing `DbContext` Lifetime:** The Unit of Work can manage the lifetime of the `DbContext` within the scope of a unit of work.

## **Basic Implementation with EF Core:**

1. **Define the Unit of Work Interface:**
    
    ```csharp
    public interface IUnitOfWork : IDisposable
    {
        IBlogRepository Blogs { get; }
        IPostRepository Posts { get; } // If you have multiple repositories
        Task<int> SaveChangesAsync();
    }
    ```
    
2. **Implement the Unit of Work (using EF Core `DbContext`):**C#
    
    ```csharp
    public class UnitOfWork : IUnitOfWork
    {
        private readonly BloggingContext _dbContext;
        private BlogRepository _blogRepository;
        private PostRepository _postRepository;
    
        public UnitOfWork(BloggingContext dbContext)
        {
            _dbContext = dbContext;
        }
    
        public IBlogRepository Blogs
        {
            get
            {
                if (_blogRepository == null)
                {
                    _blogRepository = new BlogRepository(_dbContext);
                }
                return _blogRepository;
            }
        }
    
        public IPostRepository Posts
        {
            get
            {
                if (_postRepository == null)
                {
                    _postRepository = new PostRepository(_dbContext);
                }
                return _postRepository;
            }
        }
    
        public async Task<int> SaveChangesAsync()
        {
            return await _dbContext.SaveChangesAsync();
        }
    
        private bool _disposed = false;
    
        protected virtual void Dispose(bool disposing)
        {
            if (!_disposed)
            {
                if (disposing)
                {
                    _dbContext.Dispose();
                }
            }
            _disposed = true;
        }
    
        public void Dispose()
        {
            Dispose(true);
            GC.SuppressFinalize(this);
        }
    }
    ```
    
3. **Use the Unit of Work in a Service or Controller:**
    
    ```csharp
    public class OrderService
    {
        private readonly IUnitOfWork _unitOfWork;
    
        public OrderService(IUnitOfWork unitOfWork)
        {
            _unitOfWork = unitOfWork;
        }
    
        public async Task ProcessOrderAsync(int productId, int quantity, int customerId)
        {
            // Get the product from the repository
            var product = await _unitOfWork.Products.GetByIdAsync(productId);
            if (product == null || product.Stock < quantity)
            {
                throw new InvalidOperationException("Not enough stock.");
            }
    
            // Create the order
            var order = new Order { CustomerId = customerId, OrderDate = DateTime.UtcNow };
            await _unitOfWork.Orders.AddAsync(order);
            await _unitOfWork.SaveChangesAsync(); // Save the order to get the OrderId
    
            // Create the order detail
            var orderDetail = new OrderDetail
            {
                OrderId = order.OrderId,
                ProductId = productId,
                Quantity = quantity,
                UnitPrice = product.Price
            };
            await _unitOfWork.OrderDetails.AddAsync(orderDetail);
    
            // Update the product stock
            product.Stock -= quantity;
            _unitOfWork.Products.UpdateAsync(product);
    
            // Save all changes in a single transaction
            await _unitOfWork.SaveChangesAsync();
        }
    }
    ```
    

---

# **Benefits of Implementing Repository and Unit of Work with EF Core**

- **Cleaner and More Maintainable Code:** Separating business logic and data access logic makes the code easier to read, understand, and maintain. When changes occur in the data layer, you only need to modify the repositories.
- **Improved Testability:** You can easily create mock implementations of the repository and unit of work interfaces to test services or controllers independently of the actual database. This helps write faster and more reliable unit tests.
- **Reduced Dependency on EF Core:** Your business logic doesn't directly depend on `DbContext` or the specifics of EF Core. This makes it easier to change the ORM in the future if needed.
- **More Effective Transaction Management:** The Unit of Work helps you manage transactions explicitly, ensuring data consistency when performing multiple related operations.
- **Code Reusability:** Common data access operations implemented in the repository can be reused in various parts of the application.
- **Better Application Structure:** Using these patterns helps your application have a clearer structure and adhere to good design principles.

## **Considerations When Implementing:**

- **Increased Complexity:** Adding repository and unit of work classes will increase the number of classes in your project. For small applications, it might not be necessary.
- **Consider the Scope of the Unit of Work:** Clearly define the scope of a "unit of work" (e.g., a request in a web application).
- **Dependency Injection:** Use Dependency Injection (DI) to manage the lifecycle and provide instances of repositories and the unit of work to services and controllers. In ASP.NET Core, this is built-in.

---

# DbContext vs Unit of Work Patterns

At first glance, the Unit of Work pattern can seem very similar to the `DbContext` in EF Core, and in simple scenarios, they might even appear to serve similar purposes. However, there are key differences in their responsibilities, scope, and the benefits they provide, especially in more complex applications. Let's break down the distinctions:

## **`DbContext` in EF Core:**

- **Primary Responsibility:** The `DbContext` primarily represents a **session with the database**. Its core responsibilities include:
    - **Tracking Changes:** It keeps track of the changes made to the entities it retrieves.
    - **Querying:** It provides the mechanism to query data from the database using LINQ to Entities.
    - **Saving Changes:** It's responsible for persisting the tracked changes to the database via the `SaveChanges()` or `SaveChangesAsync()` methods.
    - **Configuration:** It's used to configure the database connection and the mapping between your entities and the database schema (using `OnConfiguring` and `OnModelCreating`).
    - **Identity Management:** It manages the identity of the entities it tracks.
- **Scope:** Typically, the scope of a `DbContext` instance is intended to be **short-lived**, often per request in a web application or within a specific logical operation. Creating a long-lived `DbContext` can lead to issues with caching stale data and increased memory consumption due to tracking a large number of entities.
- **Focus:** Its focus is on the **technical interaction with the database**.

## **Unit of Work Pattern:**

- **Primary Responsibility:** The Unit of Work pattern focuses on managing a **business transaction** as a single atomic operation. Its core responsibilities include:
    - **Coordinating Multiple Repositories:** It can orchestrate operations across multiple repositories within a single business transaction.
    - **Tracking Changes Across Repositories:** While the underlying change tracking is still done by the `DbContext` instances managed by the repositories, the Unit of Work ensures that all changes within the business transaction are committed together.
    - **Managing the Lifetime of `DbContext` Instances (Optional but Common):** The Unit of Work can control the scope and lifetime of one or more `DbContext` instances used by the repositories it manages.
    - **Providing a Single Point for Committing or Rolling Back Changes:** It typically exposes a `Commit()` or `SaveChangesAsync()` method that persists all the changes tracked by the underlying `DbContext` instances within the unit of work. If an error occurs during the business transaction, the Unit of Work can handle rollback (though EF Core's transaction support is the underlying mechanism).
- **Scope:** The scope of a Unit of Work is typically aligned with a **business transaction**. This might span multiple operations involving different entities and repositories. For example, processing an order might involve creating a new order, updating inventory, and creating invoice details – all within a single Unit of Work.
- **Focus:** Its focus is on the **business transaction and ensuring data consistency** across multiple data access operations. It provides a higher-level abstraction over the individual database interactions managed by the repositories and `DbContext`.

## Key Differences Summarized:

| Feature | `DbContext` (EF Core) | Unit of Work Pattern |
| --- | --- | --- |
| **Primary Role** | Session with the database, change tracking, querying, saving | Managing a business transaction as a single atomic unit |
| **Scope** | Typically short-lived, often per request/operation | Aligned with a business transaction, potentially longer |
| **Manages** | Entities tracked within its context | Operations across multiple repositories, `DbContext` lifetime (potentially) |
| **Commitment** | `SaveChanges()` on the `DbContext` instance | `Commit()` or `SaveChangesAsync()` on the Unit of Work, orchestrating `DbContext` saves |
| **Abstraction Level** | Lower-level, directly interacts with the database | Higher-level, abstracts business transactions over data access |
| **Purpose** | Technical interaction with the database | Ensuring data consistency and managing business transactions |

## **Analogy:**

Think of `DbContext` as a single worker who knows how to interact with the database (read, write, update). The Unit of Work is like a project manager who oversees multiple workers (repositories) to complete a specific business task (transaction). The project manager ensures all the workers complete their assigned tasks successfully, and if any worker fails, the entire task is considered incomplete.

### **Benefits of Using Unit of Work with EF Core (even though you have `DbContext`):**

- **Improved Transaction Management:** Explicitly defines the scope of a business transaction, especially when it involves multiple repositories and entities.
- **Better Separation of Concerns:** Keeps the transaction management logic separate from the individual repository implementations.
- **Easier Testing of Business Logic:** You can mock the Unit of Work to test business services that perform multiple data operations without needing to set up complex mock repositories and `DbContext` interactions.
- **Reduced Boilerplate Code:** Centralizes the `SaveChanges()` call for a business transaction, reducing repetitive code in your services.
- **Enhanced Data Consistency:** Makes it clearer when a set of operations should be treated as a single atomic unit.