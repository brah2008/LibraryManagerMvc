# LibraryManagerMvc
Library Manager ASP.Net Core Web App Mvc

Creating a complete project involves multiple files and components, so I'll provide you with a basic structure for a Library Manager project in C# ASP.NET Core, both for the web app MVC and the web API. This example assumes you have some familiarity with ASP.NET Core MVC and Web API.
**Library Manager - ASP.NET Core MVC Web App:**
1. Create a new ASP.NET Core MVC project using Visual Studio or the command line:
   ```bash
   dotnet new mvc -n LibraryManagerMvc
   cd LibraryManagerMvc
   ```
2. Define a model for the library, e.g., `Book.cs`:
   ```csharp
   public class Book
   {
       public int Id { get; set; }
       public string Title { get; set; }
       public string Author { get; set; }
   }
   ```
3. Create a `LibraryService.cs` to handle data operations:
   ```csharp
   public class LibraryService
   {
       private List<Book> _books = new List<Book>();
       public IEnumerable<Book> GetAllBooks()
       {
           return _books;
       }
       public void AddBook(Book book)
       {
           _books.Add(book);
       }
   }
   ```
4. In `Startup.cs`, configure services and add controllers:
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddSingleton<LibraryService>();
       services.AddControllersWithViews();
   }
   ```
5. Create a `BooksController.cs` for handling book-related actions.
**Library Manager - ASP.NET Core Web API:**
1. Create a new ASP.NET Core Web API project:
   ```bash
   dotnet new webapi -n LibraryManagerApi
   cd LibraryManagerApi
   ```
2. Use the same `Book.cs` model and `LibraryService.cs` from the MVC project.
3. In `Startup.cs`, configure services and add controllers:
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddSingleton<LibraryService>();
       services.AddControllers();
   }
   ```
4. Create a `BooksController.cs` for handling book-related API actions.
Here's a simple example for both controllers:
```csharp
[ApiController]
[Route("api/[controller]")]
public class BooksController : ControllerBase
{
    private readonly LibraryService _libraryService;
    public BooksController(LibraryService libraryService)
    {
        _libraryService = libraryService;
    }
    [HttpGet]
    public IEnumerable<Book> GetAllBooks()
    {
        return _libraryService.GetAllBooks();
    }
    [HttpPost]
    public IActionResult AddBook(Book book)
    {
        _libraryService.AddBook(book);
        return Ok();
    }
}
```
These are very basic examples. In a real-world application, you'd likely use a database, handle validation, authentication, authorization, and more. Ensure you've added the necessary packages using NuGet for these functionalities.
Remember, this is just a starting point, and you may need to modify and extend the project based on your specific requirements.


Certainly! Let's continue building upon the Library Manager project. We'll add a simple database using Entity Framework Core for both the MVC and Web API projects. Additionally, we'll enhance error handling, validation, and implement basic views for the MVC project.
**Library Manager - ASP.NET Core MVC Web App:**
6. Install Entity Framework Core and SQLite packages:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.Sqlite
   dotnet add package Microsoft.EntityFrameworkCore.Design
   ```
7. Modify `Book.cs` to include annotations for validation:
   ```csharp
   using System.ComponentModel.DataAnnotations;
   public class Book
   {
       public int Id { get; set; }
       [Required(ErrorMessage = "Title is required.")]
       public string Title { get; set; }
       [Required(ErrorMessage = "Author is required.")]
       public string Author { get; set; }
   }
   ```
8. Update `LibraryService.cs` to use Entity Framework Core:
   ```csharp
   public class LibraryService
   {
       private readonly ApplicationDbContext _context;
       public LibraryService(ApplicationDbContext context)
       {
           _context = context;
       }
       public IEnumerable<Book> GetAllBooks()
       {
           return _context.Books.ToList();
       }
       public void AddBook(Book book)
       {
           _context.Books.Add(book);
           _context.SaveChanges();
       }
   }
   ```
9. Create `ApplicationDbContext.cs` for database context:
   ```csharp
   using Microsoft.EntityFrameworkCore;
   public class ApplicationDbContext : DbContext
   {
       public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
           : base(options)
       {
       }
       public DbSet<Book> Books { get; set; }
   }
   ```
10. Update `Startup.cs` to configure the database and use dependency injection:
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddDbContext<ApplicationDbContext>(options =>
           options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));
       services.AddScoped<LibraryService>();
       services.AddControllersWithViews();
   }
   ```
11. Create and apply database migration:
    ```bash
    dotnet ef migrations add InitialCreate
    dotnet ef database update
    ```
12. Update `BooksController.cs` to use the `LibraryService` with dependency injection:
   ```csharp
   public class BooksController : Controller
   {
       private readonly LibraryService _libraryService;
       public BooksController(LibraryService libraryService)
       {
           _libraryService = libraryService;
       }
       public IActionResult Index()
       {
           var books = _libraryService.GetAllBooks();
           return View(books);
       }
       [HttpPost]
       public IActionResult AddBook(Book book)
       {
           if (ModelState.IsValid)
           {
               _libraryService.AddBook(book);
               return RedirectToAction("Index");
           }
           return View("Index", _libraryService.GetAllBooks());
       }
   }
   ```
**Library Manager - ASP.NET Core Web API:**
5. Install Entity Framework Core and SQLite packages:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.Sqlite
   dotnet add package Microsoft.EntityFrameworkCore.Design
   ```
6. Follow steps 7-11 from the MVC project to update the `Book` model, `LibraryService`, `ApplicationDbContext`, configure services, and apply database migration.
7. Update `BooksController.cs` to use the `LibraryService` with dependency injection:
   ```csharp
   [ApiController]
   [Route("api/[controller]")]
   public class BooksController : ControllerBase
   {
       private readonly LibraryService _libraryService;
       public BooksController(LibraryService libraryService)
       {
           _libraryService = libraryService;
       }
       [HttpGet]
       public IActionResult GetAllBooks()
       {
           var books = _libraryService.GetAllBooks();
           return Ok(books);
       }
       [HttpPost]
       public IActionResult AddBook(Book book)
       {
           if (ModelState.IsValid)
           {
               _libraryService.AddBook(book);
               return Ok();
           }
           return BadRequest(ModelState);
       }
   }
   ```
Remember to run your applications (`dotnet run`) and test the API and MVC routes to ensure everything is working as expected.


Certainly! Let's continue with the enhancements to the Library Manager project. In this section, we'll add basic views to the MVC project for listing books and adding new books.
**Library Manager - ASP.NET Core MVC Web App:**
13. Create `Views/Books` folder in your `LibraryManagerMvc` project.
14. Add a new file `Index.cshtml` inside the `Views/Books` folder:
```html
@model IEnumerable<Book>
<h2>Book List</h2>
<table class="table">
    <thead>
        <tr>
            <th>ID</th>
            <th>Title</th>
            <th>Author</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var book in Model)
        {
            <tr>
                <td>@book.Id</td>
                <td>@book.Title</td>
                <td>@book.Author</td>
            </tr>
        }
    </tbody>
</table>
<h2>Add New Book</h2>
@using (Html.BeginForm("AddBook", "Books", FormMethod.Post))
{
    <div class="form-group">
        <label asp-for="Title"></label>
        <input asp-for="Title" class="form-control" />
        <span asp-validation-for="Title" class="text-danger"></span>
    </div>
    <div class="form-group">
        <label asp-for="Author"></label>
        <input asp-for="Author" class="form-control" />
        <span asp-validation-for="Author" class="text-danger"></span>
    </div>
    <button type="submit" class="btn btn-primary">Add Book</button>
}
```
15. Update `BooksController.cs` to include the action methods for rendering views:
```csharp
public class BooksController : Controller
{
    private readonly LibraryService _libraryService;
    public BooksController(LibraryService libraryService)
    {
        _libraryService = libraryService;
    }
    public IActionResult Index()
    {
        var books = _libraryService.GetAllBooks();
        return View(books);
    }
    [HttpPost]
    public IActionResult AddBook(Book book)
    {
        if (ModelState.IsValid)
        {
            _libraryService.AddBook(book);
            return RedirectToAction("Index");
        }
        var books = _libraryService.GetAllBooks();
        return View("Index", books);
    }
}
```
16. Run your MVC project (`dotnet run`), navigate to `/Books`, and you should see a list of books and a form to add new books.
**Note:** For the web API project, since it's primarily meant for data access, we won't be adding views. You can test the API endpoints using tools like Postman or Swagger.
With these additions, you've created a basic MVC web app with views to list and add books. Keep in mind that this is a simple example, and in a real-world application, you would likely implement more features, including user authentication, detailed error handling, and more sophisticated UI components.


Certainly! Let's continue enhancing the Library Manager project by adding user authentication and refining error handling in both the MVC web app and the Web API.
**Library Manager - ASP.NET Core MVC Web App:**
17. Install the necessary packages for authentication:
   ```bash
   dotnet add package Microsoft.AspNetCore.Authentication.Cookies
   dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
   ```
18. Update `Startup.cs` to configure authentication and identity services:
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       // ... (existing configurations)
       services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
           .AddCookie(options =>
           {
               options.Cookie.Name = "LibraryManagerCookie";
               options.LoginPath = "/Account/Login";
               options.AccessDeniedPath = "/Account/AccessDenied";
           });
       services.AddIdentity<IdentityUser, IdentityRole>()
           .AddEntityFrameworkStores<ApplicationDbContext>()
           .AddDefaultTokenProviders();
       services.AddScoped<LibraryService>();
       services.AddControllersWithViews();
   }
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
       // ... (existing configurations)
       app.UseAuthentication();
       app.UseAuthorization();
       // ... (remaining configurations)
   }
   ```
19. Create an `AccountController.cs` for handling user authentication:
   ```csharp
   public class AccountController : Controller
   {
       private readonly SignInManager<IdentityUser> _signInManager;
       public AccountController(SignInManager<IdentityUser> signInManager)
       {
           _signInManager = signInManager;
       }
       [HttpGet]
       public IActionResult Login()
       {
           return View();
       }
       [HttpPost]
       public async Task<IActionResult> Login(string returnUrl, string username, string password)
       {
           var result = await _signInManager.PasswordSignInAsync(username, password, false, false);
           if (result.Succeeded)
           {
               return Redirect(returnUrl ?? "/");
           }
           ModelState.AddModelError(string.Empty, "Invalid login attempt");
           return View();
       }
       [HttpPost]
       public async Task<IActionResult> Logout()
       {
           await _signInManager.SignOutAsync();
           return RedirectToAction("Index", "Home");
       }
   }
   ```
20. Update the `BooksController.cs` to include the `[Authorize]` attribute on actions that require authentication:
   ```csharp
   [Authorize]
   public class BooksController : Controller
   {
       // ... (existing code)
       [HttpPost]
       public IActionResult AddBook(Book book)
       {
           if (ModelState.IsValid)
           {
               _libraryService.AddBook(book);
               return RedirectToAction("Index");
           }
           var books = _libraryService.GetAllBooks();
           return View("Index", books);
       }
   }
   ```
21. Create a `Views/Account` folder and add a `Login.cshtml` file for the login view.
**Library Manager - ASP.NET Core Web API:**
22. Update `BooksController.cs` in the API project to include basic error handling:
   ```csharp
   [ApiController]
   [Route("api/[controller]")]
   public class BooksController : ControllerBase
   {
       private readonly LibraryService _libraryService;
       public BooksController(LibraryService libraryService)
       {
           _libraryService = libraryService;
       }
       [HttpGet]
       public IActionResult GetAllBooks()
       {
           try
           {
               var books = _libraryService.GetAllBooks();
               return Ok(books);
           }
           catch (Exception ex)
           {
               return StatusCode(500, $"Internal server error: {ex.Message}");
           }
       }
       [HttpPost]
       public IActionResult AddBook(Book book)
       {
           try
           {
               if (ModelState.IsValid)
               {
                   _libraryService.AddBook(book);
                   return Ok();
               }
               return BadRequest(ModelState);
           }
           catch (Exception ex)
           {
               return StatusCode(500, $"Internal server error: {ex.Message}");
           }
       }
   }
   ```
23. Test the authentication in the MVC app by running it (`dotnet run`) and navigating to `/Account/Login`. For the API, you can use tools like Postman or Swagger.
With these additions, you've implemented basic user authentication in the MVC app and enhanced error handling in both the MVC app and the Web API. Keep in mind that this is a simple example, and in a real-world application, you would likely implement more features, including user roles, registration, password recovery, and more advanced error handling.

It looks like the instructions were cut off at the end. Let me continue from where it left off:
**Library Manager - ASP.NET Core MVC Web App:**
21. Create a `Views/Account` folder and add a `Login.cshtml` file for the login view:
   ```html
   @model LoginViewModel
   <h2>Login</h2>
   <form asp-controller="Account" asp-action="Login" method="post">
       <div asp-validation-summary="All" class="text-danger"></div>
       <div class="form-group">
           <label asp-for="Username"></label>
           <input asp-for="Username" class="form-control" />
           <span asp-validation-for="Username" class="text-danger"></span>
       </div>
       <div class="form-group">
           <label asp-for="Password"></label>
           <input asp-for="Password" type="password" class="form-control" />
           <span asp-validation-for="Password" class="text-danger"></span>
       </div>
       <button type="submit" class="btn btn-primary">Login</button>
   </form>
   ```
   22. Update the `LoginViewModel.cs` model:
   ```csharp
   public class LoginViewModel
   {
       [Required(ErrorMessage = "Username is required.")]
       public string Username { get; set; }
       [Required(ErrorMessage = "Password is required.")]
       public string Password { get; set; }
   }
   ```
   23. Update the `AccountController.cs` to use the `LoginViewModel`:
   ```csharp
   [HttpGet]
   public IActionResult Login()
   {
       return View(new LoginViewModel());
   }
   [HttpPost]
   public async Task<IActionResult> Login(string returnUrl, LoginViewModel model)
   {
       if (ModelState.IsValid)
       {
           var result = await _signInManager.PasswordSignInAsync(model.Username, model.Password, false, false);
           if (result.Succeeded)
           {
               return Redirect(returnUrl ?? "/");
           }
           ModelState.AddModelError(string.Empty, "Invalid login attempt");
       }
       return View(model);
   }
   ```
   Now, when you run your MVC app and navigate to `/Account/Login`, you should see a login form.
**Library Manager - ASP.NET Core Web API:**
24. For the Web API project, continue testing the API endpoints using tools like Postman or Swagger.
With these additions, you've implemented basic user authentication in the MVC app with a login view and enhanced error handling. Remember, this is a simplified example, and in a production environment, you would implement more robust security measures and refine the user authentication flow.


# LibraryManagerMvc
Library Manager ASP.Net Core Web App Mvc

Creating a complete project involves multiple files and components, so I'll provide you with a basic structure for a Library Manager project in C# ASP.NET Core, both for the web app MVC and the web API. This example assumes you have some familiarity with ASP.NET Core MVC and Web API.
**Library Manager - ASP.NET Core MVC Web App:**
1. Create a new ASP.NET Core MVC project using Visual Studio or the command line:
   ```bash
   dotnet new mvc -n LibraryManagerMvc
   cd LibraryManagerMvc
   ```
2. Define a model for the library, e.g., `Book.cs`:
   ```csharp
   public class Book
   {
       public int Id { get; set; }
       public string Title { get; set; }
       public string Author { get; set; }
   }
   ```
3. Create a `LibraryService.cs` to handle data operations:
   ```csharp
   public class LibraryService
   {
       private List<Book> _books = new List<Book>();
       public IEnumerable<Book> GetAllBooks()
       {
           return _books;
       }
       public void AddBook(Book book)
       {
           _books.Add(book);
       }
   }
   ```
4. In `Startup.cs`, configure services and add controllers:
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddSingleton<LibraryService>();
       services.AddControllersWithViews();
   }
   ```
5. Create a `BooksController.cs` for handling book-related actions.
**Library Manager - ASP.NET Core Web API:**
1. Create a new ASP.NET Core Web API project:
   ```bash
   dotnet new webapi -n LibraryManagerApi
   cd LibraryManagerApi
   ```
2. Use the same `Book.cs` model and `LibraryService.cs` from the MVC project.
3. In `Startup.cs`, configure services and add controllers:
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddSingleton<LibraryService>();
       services.AddControllers();
   }
   ```
4. Create a `BooksController.cs` for handling book-related API actions.
Here's a simple example for both controllers:
```csharp
[ApiController]
[Route("api/[controller]")]
public class BooksController : ControllerBase
{
    private readonly LibraryService _libraryService;
    public BooksController(LibraryService libraryService)
    {
        _libraryService = libraryService;
    }
    [HttpGet]
    public IEnumerable<Book> GetAllBooks()
    {
        return _libraryService.GetAllBooks();
    }
    [HttpPost]
    public IActionResult AddBook(Book book)
    {
        _libraryService.AddBook(book);
        return Ok();
    }
}
```
These are very basic examples. In a real-world application, you'd likely use a database, handle validation, authentication, authorization, and more. Ensure you've added the necessary packages using NuGet for these functionalities.
Remember, this is just a starting point, and you may need to modify and extend the project based on your specific requirements.


Certainly! Let's continue building upon the Library Manager project. We'll add a simple database using Entity Framework Core for both the MVC and Web API projects. Additionally, we'll enhance error handling, validation, and implement basic views for the MVC project.
**Library Manager - ASP.NET Core MVC Web App:**
6. Install Entity Framework Core and SQLite packages:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.Sqlite
   dotnet add package Microsoft.EntityFrameworkCore.Design
   ```
7. Modify `Book.cs` to include annotations for validation:
   ```csharp
   using System.ComponentModel.DataAnnotations;
   public class Book
   {
       public int Id { get; set; }
       [Required(ErrorMessage = "Title is required.")]
       public string Title { get; set; }
       [Required(ErrorMessage = "Author is required.")]
       public string Author { get; set; }
   }
   ```
8. Update `LibraryService.cs` to use Entity Framework Core:
   ```csharp
   public class LibraryService
   {
       private readonly ApplicationDbContext _context;
       public LibraryService(ApplicationDbContext context)
       {
           _context = context;
       }
       public IEnumerable<Book> GetAllBooks()
       {
           return _context.Books.ToList();
       }
       public void AddBook(Book book)
       {
           _context.Books.Add(book);
           _context.SaveChanges();
       }
   }
   ```
9. Create `ApplicationDbContext.cs` for database context:
   ```csharp
   using Microsoft.EntityFrameworkCore;
   public class ApplicationDbContext : DbContext
   {
       public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
           : base(options)
       {
       }
       public DbSet<Book> Books { get; set; }
   }
   ```
10. Update `Startup.cs` to configure the database and use dependency injection:
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddDbContext<ApplicationDbContext>(options =>
           options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));
       services.AddScoped<LibraryService>();
       services.AddControllersWithViews();
   }
   ```
11. Create and apply database migration:
    ```bash
    dotnet ef migrations add InitialCreate
    dotnet ef database update
    ```
12. Update `BooksController.cs` to use the `LibraryService` with dependency injection:
   ```csharp
   public class BooksController : Controller
   {
       private readonly LibraryService _libraryService;
       public BooksController(LibraryService libraryService)
       {
           _libraryService = libraryService;
       }
       public IActionResult Index()
       {
           var books = _libraryService.GetAllBooks();
           return View(books);
       }
       [HttpPost]
       public IActionResult AddBook(Book book)
       {
           if (ModelState.IsValid)
           {
               _libraryService.AddBook(book);
               return RedirectToAction("Index");
           }
           return View("Index", _libraryService.GetAllBooks());
       }
   }
   ```
**Library Manager - ASP.NET Core Web API:**
5. Install Entity Framework Core and SQLite packages:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.Sqlite
   dotnet add package Microsoft.EntityFrameworkCore.Design
   ```
6. Follow steps 7-11 from the MVC project to update the `Book` model, `LibraryService`, `ApplicationDbContext`, configure services, and apply database migration.
7. Update `BooksController.cs` to use the `LibraryService` with dependency injection:
   ```csharp
   [ApiController]
   [Route("api/[controller]")]
   public class BooksController : ControllerBase
   {
       private readonly LibraryService _libraryService;
       public BooksController(LibraryService libraryService)
       {
           _libraryService = libraryService;
       }
       [HttpGet]
       public IActionResult GetAllBooks()
       {
           var books = _libraryService.GetAllBooks();
           return Ok(books);
       }
       [HttpPost]
       public IActionResult AddBook(Book book)
       {
           if (ModelState.IsValid)
           {
               _libraryService.AddBook(book);
               return Ok();
           }
           return BadRequest(ModelState);
       }
   }
   ```
Remember to run your applications (`dotnet run`) and test the API and MVC routes to ensure everything is working as expected.


Certainly! Let's continue with the enhancements to the Library Manager project. In this section, we'll add basic views to the MVC project for listing books and adding new books.
**Library Manager - ASP.NET Core MVC Web App:**
13. Create `Views/Books` folder in your `LibraryManagerMvc` project.
14. Add a new file `Index.cshtml` inside the `Views/Books` folder:
```html
@model IEnumerable<Book>
<h2>Book List</h2>
<table class="table">
    <thead>
        <tr>
            <th>ID</th>
            <th>Title</th>
            <th>Author</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var book in Model)
        {
            <tr>
                <td>@book.Id</td>
                <td>@book.Title</td>
                <td>@book.Author</td>
            </tr>
        }
    </tbody>
</table>
<h2>Add New Book</h2>
@using (Html.BeginForm("AddBook", "Books", FormMethod.Post))
{
    <div class="form-group">
        <label asp-for="Title"></label>
        <input asp-for="Title" class="form-control" />
        <span asp-validation-for="Title" class="text-danger"></span>
    </div>
    <div class="form-group">
        <label asp-for="Author"></label>
        <input asp-for="Author" class="form-control" />
        <span asp-validation-for="Author" class="text-danger"></span>
    </div>
    <button type="submit" class="btn btn-primary">Add Book</button>
}
```
15. Update `BooksController.cs` to include the action methods for rendering views:
```csharp
public class BooksController : Controller
{
    private readonly LibraryService _libraryService;
    public BooksController(LibraryService libraryService)
    {
        _libraryService = libraryService;
    }
    public IActionResult Index()
    {
        var books = _libraryService.GetAllBooks();
        return View(books);
    }
    [HttpPost]
    public IActionResult AddBook(Book book)
    {
        if (ModelState.IsValid)
        {
            _libraryService.AddBook(book);
            return RedirectToAction("Index");
        }
        var books = _libraryService.GetAllBooks();
        return View("Index", books);
    }
}
```
16. Run your MVC project (`dotnet run`), navigate to `/Books`, and you should see a list of books and a form to add new books.
**Note:** For the web API project, since it's primarily meant for data access, we won't be adding views. You can test the API endpoints using tools like Postman or Swagger.
With these additions, you've created a basic MVC web app with views to list and add books. Keep in mind that this is a simple example, and in a real-world application, you would likely implement more features, including user authentication, detailed error handling, and more sophisticated UI components.


Certainly! Let's continue enhancing the Library Manager project by adding user authentication and refining error handling in both the MVC web app and the Web API.
**Library Manager - ASP.NET Core MVC Web App:**
17. Install the necessary packages for authentication:
   ```bash
   dotnet add package Microsoft.AspNetCore.Authentication.Cookies
   dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
   ```
18. Update `Startup.cs` to configure authentication and identity services:
   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       // ... (existing configurations)
       services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
           .AddCookie(options =>
           {
               options.Cookie.Name = "LibraryManagerCookie";
               options.LoginPath = "/Account/Login";
               options.AccessDeniedPath = "/Account/AccessDenied";
           });
       services.AddIdentity<IdentityUser, IdentityRole>()
           .AddEntityFrameworkStores<ApplicationDbContext>()
           .AddDefaultTokenProviders();
       services.AddScoped<LibraryService>();
       services.AddControllersWithViews();
   }
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
       // ... (existing configurations)
       app.UseAuthentication();
       app.UseAuthorization();
       // ... (remaining configurations)
   }
   ```
19. Create an `AccountController.cs` for handling user authentication:
   ```csharp
   public class AccountController : Controller
   {
       private readonly SignInManager<IdentityUser> _signInManager;
       public AccountController(SignInManager<IdentityUser> signInManager)
       {
           _signInManager = signInManager;
       }
       [HttpGet]
       public IActionResult Login()
       {
           return View();
       }
       [HttpPost]
       public async Task<IActionResult> Login(string returnUrl, string username, string password)
       {
           var result = await _signInManager.PasswordSignInAsync(username, password, false, false);
           if (result.Succeeded)
           {
               return Redirect(returnUrl ?? "/");
           }
           ModelState.AddModelError(string.Empty, "Invalid login attempt");
           return View();
       }
       [HttpPost]
       public async Task<IActionResult> Logout()
       {
           await _signInManager.SignOutAsync();
           return RedirectToAction("Index", "Home");
       }
   }
   ```
20. Update the `BooksController.cs` to include the `[Authorize]` attribute on actions that require authentication:
   ```csharp
   [Authorize]
   public class BooksController : Controller
   {
       // ... (existing code)
       [HttpPost]
       public IActionResult AddBook(Book book)
       {
           if (ModelState.IsValid)
           {
               _libraryService.AddBook(book);
               return RedirectToAction("Index");
           }
           var books = _libraryService.GetAllBooks();
           return View("Index", books);
       }
   }
   ```
21. Create a `Views/Account` folder and add a `Login.cshtml` file for the login view.
**Library Manager - ASP.NET Core Web API:**
22. Update `BooksController.cs` in the API project to include basic error handling:
   ```csharp
   [ApiController]
   [Route("api/[controller]")]
   public class BooksController : ControllerBase
   {
       private readonly LibraryService _libraryService;
       public BooksController(LibraryService libraryService)
       {
           _libraryService = libraryService;
       }
       [HttpGet]
       public IActionResult GetAllBooks()
       {
           try
           {
               var books = _libraryService.GetAllBooks();
               return Ok(books);
           }
           catch (Exception ex)
           {
               return StatusCode(500, $"Internal server error: {ex.Message}");
           }
       }
       [HttpPost]
       public IActionResult AddBook(Book book)
       {
           try
           {
               if (ModelState.IsValid)
               {
                   _libraryService.AddBook(book);
                   return Ok();
               }
               return BadRequest(ModelState);
           }
           catch (Exception ex)
           {
               return StatusCode(500, $"Internal server error: {ex.Message}");
           }
       }
   }
   ```
23. Test the authentication in the MVC app by running it (`dotnet run`) and navigating to `/Account/Login`. For the API, you can use tools like Postman or Swagger.
With these additions, you've implemented basic user authentication in the MVC app and enhanced error handling in both the MVC app and the Web API. Keep in mind that this is a simple example, and in a real-world application, you would likely implement more features, including user roles, registration, password recovery, and more advanced error handling.

It looks like the instructions were cut off at the end. Let me continue from where it left off:
**Library Manager - ASP.NET Core MVC Web App:**
21. Create a `Views/Account` folder and add a `Login.cshtml` file for the login view:
   ```html
   @model LoginViewModel
   <h2>Login</h2>
   <form asp-controller="Account" asp-action="Login" method="post">
       <div asp-validation-summary="All" class="text-danger"></div>
       <div class="form-group">
           <label asp-for="Username"></label>
           <input asp-for="Username" class="form-control" />
           <span asp-validation-for="Username" class="text-danger"></span>
       </div>
       <div class="form-group">
           <label asp-for="Password"></label>
           <input asp-for="Password" type="password" class="form-control" />
           <span asp-validation-for="Password" class="text-danger"></span>
       </div>
       <button type="submit" class="btn btn-primary">Login</button>
   </form>
   ```
   22. Update the `LoginViewModel.cs` model:
   ```csharp
   public class LoginViewModel
   {
       [Required(ErrorMessage = "Username is required.")]
       public string Username { get; set; }
       [Required(ErrorMessage = "Password is required.")]
       public string Password { get; set; }
   }
   ```
   23. Update the `AccountController.cs` to use the `LoginViewModel`:
   ```csharp
   [HttpGet]
   public IActionResult Login()
   {
       return View(new LoginViewModel());
   }
   [HttpPost]
   public async Task<IActionResult> Login(string returnUrl, LoginViewModel model)
   {
       if (ModelState.IsValid)
       {
           var result = await _signInManager.PasswordSignInAsync(model.Username, model.Password, false, false);
           if (result.Succeeded)
           {
               return Redirect(returnUrl ?? "/");
           }
           ModelState.AddModelError(string.Empty, "Invalid login attempt");
       }
       return View(model);
   }
   ```
   Now, when you run your MVC app and navigate to `/Account/Login`, you should see a login form.
**Library Manager - ASP.NET Core Web API:**
24. For the Web API project, continue testing the API endpoints using tools like Postman or Swagger.
With these additions, you've implemented basic user authentication in the MVC app with a login view and enhanced error handling. Remember, this is a simplified example, and in a production environment, you would implement more robust security measures and refine the user authentication flow.

Certainly! Let's continue with the instructions for adding user authentication to the Library Manager project.

**Library Manager - ASP.NET Core MVC Web App:**

1- Install the necessary packages for authentication:
```bash

dotnet add package Microsoft.AspNetCore.Authentication.Cookies
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
```

1- Update **Startup.cs** to configure authentication and identity services:
```csharp

public void ConfigureServices(IServiceCollection services)
{
    // ... (existing configurations)
    services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
        .AddCookie(options =>
        {
            options.Cookie.Name = "LibraryManagerCookie";
            options.LoginPath = "/Account/Login";
            options.AccessDeniedPath = "/Account/AccessDenied";
        });
    services.AddIdentity<IdentityUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();
    services.AddScoped<LibraryService>();
    services.AddControllersWithViews();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ... (existing configurations)
    app.UseAuthentication();
    app.UseAuthorization();
    // ... (remaining configurations)
}
```
1- Create an **AccountController.cs** for handling user authentication:
```csharp

public class AccountController : Controller
{
    private readonly SignInManager<IdentityUser> _signInManager;
    public AccountController(SignInManager<IdentityUser> signInManager)
    {
        _signInManager = signInManager;
    }

    [HttpGet]
    public IActionResult Login()
    {
        return View();
    }

    [HttpPost]
    public async Task<IActionResult> Login(string returnUrl, string username, string password)
    {
        var result = await _signInManager.PasswordSignInAsync(username, password, false, false);
        if (result.Succeeded)
        {
            return Redirect(returnUrl ?? "/");
        }
        ModelState.AddModelError(string.Empty, "Invalid login attempt");
        return View();
    }

    [HttpPost]
    public async Task<IActionResult> Logout()
    {
        await _signInManager.SignOutAsync();
        return RedirectToAction("Index", "Home");
    }
}
```
1- Update the **BooksController.cs** to include the **[Authorize]** attribute on actions that require authentication:
```csharp

[Authorize]
public class BooksController : Controller
{
    // ... (existing code)

    [HttpPost]
    public IActionResult AddBook(Book book)
    {
        if (ModelState.IsValid)
        {
            _libraryService.AddBook(book);
            return RedirectToAction("Index");
        }
        var books = _libraryService.GetAllBooks();
        return View("Index", books);
    }
}
```
1- Create a **Views/Account** folder and add a **Login.cshtml** file for the login view.
**Library Manager - ASP.NET Core Web API:**

2- Update **BooksController.cs** in the API project to include basic error handling:
```csharp

[ApiController]
[Route("api/[controller]")]
public class BooksController : ControllerBase
{
    private readonly LibraryService _libraryService;
    public BooksController(LibraryService libraryService)
    {
        _libraryService = libraryService;
    }

    [HttpGet]
    public IActionResult GetAllBooks()
    {
        try
        {
            var books = _libraryService.GetAllBooks();
            return Ok(books);
        }
        catch (Exception ex)
        {
            return StatusCode(500, $"Internal server error: {ex.Message}");
        }
    }

    [HttpPost]
    public IActionResult AddBook(Book book)
    {
        try
        {
            if (ModelState.IsValid)
            {
                _libraryService.AddBook(book);
                return Ok();
            }
            return BadRequest(ModelState);
        }
        catch (Exception ex)
        {
            return StatusCode(500, $"Internal server error: {ex.Message}");
        }
    }
}
```
Test the authentication in the MVC app by running it (**dotnet run**) and navigating to **/Account/Login**. For the API, you can use tools like Postman or Swagger.
With these additions, you've implemented basic user authentication in the MVC app and enhanced error handling in both the MVC app and the Web API. Keep in mind that this is a simple example, and in a production environment, you would implement more robust security measures and refine the user authentication flow.

**Library Manager - ASP.NET Core MVC Web App:**

Create a **Views/Account** folder and add a **Login.cshtml** file for the login view:
```html

@model LoginViewModel
<h2>Login</h2>
<form asp-controller="Account" asp-action="Login" method="post">
    <div asp-validation-summary="All" class="text-danger"></div>
    <div class="form-group">
        <label asp-for="Username"></label>
        <input asp-for="Username" class="form-control" />
        <span asp-validation-for="Username" class="text-danger"></span>
    </div>
    <div class="form-group">
        <label asp-for="Password"></label>
        <input asp-for="Password" type="password" class="form-control" />
        <span asp-validation-for="Password" class="text-danger"></span>
    </div>
    <button type="submit" class="btn btn-primary">Login</button>
</form>
```
1- Update the **LoginViewModel.cs** model:
```csharp

public class LoginViewModel
{
    [Required(ErrorMessage = "Username is required.")]
    public string Username { get; set; }
    [Required(ErrorMessage = "Password is required.")]
    public string Password { get; set; }
}
```
1- Update the **AccountController.cs** to use the **LoginViewModel**:
```csharp

[HttpGet]
public IActionResult Login()
{
    return View(new LoginViewModel());
}
[HttpPost]
public async Task<IActionResult> Login(string returnUrl, LoginViewModel model)
{
    if (ModelState.IsValid)
    {
        var result = await _signInManager.PasswordSignInAsync(model.Username, model.Password, false, false);
        if (result.Succeeded)
        {
            return Redirect(returnUrl ?? "/");
        }
        ModelState.AddModelError(string.Empty, "Invalid login attempt");
    }
    return View(model);
}
```
Now, when you run your MVC app and navigate to **/Account/Login**, you should see a login form.
**Library Manager - ASP.NET Core Web API:**

1- For the Web API project, continue testing the API endpoints using tools like Postman or Swagger.
With these additions, you've implemented basic user authentication in the MVC app with a login view and enhanced error handling. Remember, this is a simplified example, and in a production environment, you would implement more robust security measures and refine the user authentication flow.


Let's continue by implementing user authentication and authorization in the ASP.NET Core Web API project.

**Library Manager - ASP.NET Core Web API:**

Update **BooksController.cs** in the Web API project to include basic authentication and authorization:
```csharp

[Authorize]
[ApiController]
[Route("api/[controller]")]
public class BooksController : ControllerBase
{
    private readonly LibraryService _libraryService;

    public BooksController(LibraryService libraryService)
    {
        _libraryService = libraryService;
    }

    [HttpGet]
    public IActionResult GetAllBooks()
    {
        var books = _libraryService.GetAllBooks();
        return Ok(books);
    }

    [HttpPost]
    [Authorize(Roles = "Admin")]
    public IActionResult AddBook(Book book)
    {
        try
        {
            if (ModelState.IsValid)
            {
                _libraryService.AddBook(book);
                return Ok();
            }
            return BadRequest(ModelState);
        }
        catch (Exception ex)
        {
            return StatusCode(500, $"Internal server error: {ex.Message}");
        }
    }
}
```
1- Update **Startup.cs** in the Web API project to configure authentication and authorization:
```csharp

public void ConfigureServices(IServiceCollection services)
{
    // Existing code

    services.AddSingleton<LibraryService>();
    services.AddControllers();

    services.AddAuthentication("Bearer")
        .AddJwtBearer("Bearer", options =>
        {
            options.Authority = "https://localhost:5001"; // URL of your authentication server (IdentityServer)
            options.Audience = "LibraryManagerApi"; // API resource name
        });

    services.AddAuthorization(options =>
    {
        options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Existing code

    app.UseAuthentication();
    app.UseAuthorization();

    // Existing code
}
```
1- Ensure you have IdentityServer4 configured for user authentication and authorization. You can set up IdentityServer4 as a separate project or integrate it into your existing project.

2- Configure the API resource and client in IdentityServer4. Update **Startup.cs** in the IdentityServer project:

```csharp

public void ConfigureServices(IServiceCollection services)
{
    // Existing code

    services.AddIdentityServer()
        .AddInMemoryApiResources(Config.GetApiResources())
        .AddInMemoryClients(Config.GetClients())
        .AddDeveloperSigningCredential();

    // Existing code
}
```
Create a **Config** class to define API resources and clients:

```csharp

public static class Config
{
    public static IEnumerable<ApiResource> GetApiResources()
    {
        return new List<ApiResource>
        {
            new ApiResource("LibraryManagerApi", "Library Manager API")
        };
    }

    public static IEnumerable<Client> GetClients()
    {
        return new List<Client>
        {
            new Client
            {
                ClientId = "LibraryManagerClient",
                AllowedGrantTypes = GrantTypes.ClientCredentials,
                ClientSecrets = { new Secret("LibraryManagerSecret".Sha256()) },
                AllowedScopes = { "LibraryManagerApi" }
            }
        };
    }
}
```
1- Update your client application (MVC project) to obtain and send a valid access token during API requests. You can use a library like **IdentityModel** to handle this. Install the package:
```bash

dotnet add package IdentityModel
```
Then, update your MVC project to obtain and use the access token when calling the Web API.

These additions include adding authentication and authorization to the ASP.NET Core Web API project using JWT (JSON Web Tokens) and integrating IdentityServer4 for user authentication and authorization. Ensure to configure IdentityServer4 properly and test the authentication and authorization features in your API.


Certainly! Let's continue by enhancing the Web API project to validate and enforce access control using IdentityServer4.

**Library Manager - ASP.NET Core Web API:**

1- Install the necessary packages for integrating IdentityServer4 with your Web API:

```bash

dotnet add package IdentityServer4.AccessTokenValidation
```
2- In **Startup.cs** of the Web API project, configure IdentityServer4 authentication:

```csharp

public void ConfigureServices(IServiceCollection services)
{
    // Existing code

    services.AddAuthentication("Bearer")
        .AddIdentityServerAuthentication(options =>
        {
            options.Authority = "https://localhost:5001"; // URL of your IdentityServer
            options.ApiName = "LibraryManagerApi"; // API resource name
        });

    // Existing code
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Existing code

    app.UseAuthentication();
    app.UseAuthorization();

    // Existing code
}
```
3- Update **BooksController.cs** in the Web API project to include authorization attributes:

```csharp

[ApiController]
[Route("api/[controller]")]
[Authorize]
public class BooksController : ControllerBase
{
    private readonly LibraryService _libraryService;

    public BooksController(LibraryService libraryService)
    {
        _libraryService = libraryService;
    }

    [HttpGet]
    public IActionResult GetAllBooks()
    {
        var books = _libraryService.GetAllBooks();
        return Ok(books);
    }

    [HttpPost]
    [Authorize(Roles = "Admin")] // Add role-based authorization as needed
    public IActionResult AddBook(Book book)
    {
        if (ModelState.IsValid)
        {
            _libraryService.AddBook(book);
            return Ok();
        }
        return BadRequest(ModelState);
    }
}
```
4- Optionally, you can add user roles when seeding data in **ApplicationDbContext.cs**:

```csharp

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        var adminRoleId = "admin-role-id";
        var userRoleId = "user-role-id";

        modelBuilder.Entity<IdentityRole>().HasData(
            new IdentityRole { Id = adminRoleId, Name = "Admin", NormalizedName = "ADMIN" },
            new IdentityRole { Id = userRoleId, Name = "User", NormalizedName = "USER" }
        );

        var adminUserId = "admin-user-id";
        var normalUserId = "normal-user-id";

        modelBuilder.Entity<IdentityUser>().HasData(
            new IdentityUser { Id = adminUserId, UserName = "admin", NormalizedUserName = "ADMIN", Email = "admin@example.com", NormalizedEmail = "ADMIN@EXAMPLE.COM", EmailConfirmed = true, PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Admin@123") },
            new IdentityUser { Id = normalUserId, UserName = "user", NormalizedUserName = "USER", Email = "user@example.com", NormalizedEmail = "USER@EXAMPLE.COM", EmailConfirmed = true, PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "User@123") }
        );

        modelBuilder.Entity<IdentityUserRole<string>>().HasData(
            new IdentityUserRole<string> { RoleId = adminRoleId, UserId = adminUserId },
            new IdentityUserRole<string> { RoleId = userRoleId, UserId = normalUserId }
        );
    }

    // Existing code
}
```
Run your IdentityServer4, Web API, and MVC projects. Navigate to **/Books** in the MVC app, and you should be able to access API endpoints based on the configured authorization rules.

These additions include configuring IdentityServer4 authentication in the Web API project, adding authorization attributes to controller actions, and optionally seeding data with roles for testing access control. Ensure that your IdentityServer4, Web API, and MVC projects are running, and the configuration is consistent across all projects.



Certainly! Let's continue by enhancing the MVC Web App to integrate with IdentityServer4 for user authentication.

**Library Manager - ASP.NET Core MVC Web App:**

Install the necessary packages for integrating IdentityServer4 with your MVC Web App:

```bash

dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect
dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect
```
In **Startup.cs** of the MVC project, configure OpenID Connect authentication:

```csharp

public void ConfigureServices(IServiceCollection services)
{
    // Existing code

    services.AddAuthentication(options =>
        {
            options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
            options.DefaultChallengeScheme = "oidc";
        })
        .AddCookie("Cookies")
        .AddOpenIdConnect("oidc", options =>
        {
            options.SignInScheme = "Cookies";
            options.Authority = "https://localhost:5001"; // URL of your IdentityServer
            options.ClientId = "LibraryManagerMvc"; // Client ID for your MVC app
            options.ClientSecret = "secret"; // Client secret for your MVC app
            options.ResponseType = "code";
            options.Scope.Add("openid");
            options.Scope.Add("profile");
            options.SaveTokens = true;
        });

    // Existing code
}
```
In the **AccountController.cs**, add actions for handling login and logout:

```csharp

public class AccountController : Controller
{
    private readonly IHttpClientFactory _httpClientFactory;

    public AccountController(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }

    [HttpGet]
    public IActionResult Login()
    {
        return Challenge(new AuthenticationProperties { RedirectUri = "/" }, "oidc");
    }

    [HttpPost]
    public async Task<IActionResult> Logout()
    {
        await HttpContext.SignOutAsync("Cookies");
        await HttpContext.SignOutAsync("oidc");
        return Redirect("/");
    }
}
```
Update the **BooksController.cs** in the MVC project to handle user authentication:

```csharp

public class BooksController : Controller
{
    private readonly LibraryService _libraryService;

    public BooksController(LibraryService libraryService)
    {
        _libraryService = libraryService;
    }

    [HttpGet]
    public async Task<IActionResult> Index()
    {
        var accessToken = await HttpContext.GetTokenAsync("access_token");
        var client = _httpClientFactory.CreateClient();
        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

        var response = await client.GetAsync("https://localhost:5003/api/books"); // URL of your Web API
        response.EnsureSuccessStatusCode();

        var books = await response.Content.ReadAsAsync<IEnumerable<Book>>();
        return View(books);
    }

    [HttpPost]
    public async Task<IActionResult> AddBook(Book book)
    {
        if (ModelState.IsValid)
        {
            var accessToken = await HttpContext.GetTokenAsync("access_token");
            var client = _httpClientFactory.CreateClient();
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

            var response = await client.PostAsJsonAsync("https://localhost:5003/api/books", book); // URL of your Web API
            response.EnsureSuccessStatusCode();

            return RedirectToAction("Index");
        }
        return View("Index", _libraryService.GetAllBooks());
    }
}
```
Run your IdentityServer4, Web API, and MVC projects. Navigate to **/Books** in the MVC app, and you should be prompted to log in. After logging in, you should be able to access the API endpoints.

These additions include configuring OpenID Connect authentication in the MVC project, adding actions in the **AccountController** for login and logout, and updating the **BooksController** to handle user authentication and make authenticated requests to the Web API. Ensure that your IdentityServer4, Web API, and MVC projects are running, and the configuration is consistent across all projects.


Certainly! Let's continue by enhancing the Web API to include user authentication and authorization.

**Library Manager - ASP.NET Core Web API:**

1- In the Web API project (**LibraryManagerApi**), install the necessary packages for integrating IdentityServer4 with your API:

```bash

dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```
2- In **Startup.cs** of the API project, configure JWT Bearer authentication:

```csharp

public void ConfigureServices(IServiceCollection services)
{
    // Existing code

    services.AddAuthentication("Bearer")
        .AddJwtBearer("Bearer", options =>
        {
            options.Authority = "https://localhost:5001"; // URL of your IdentityServer
            options.RequireHttpsMetadata = false; // In a production environment, set this to true
            options.Audience = "LibraryManagerApi"; // API resource name
        });

    // Existing code
}
```
3- Add the **[Authorize]** attribute to the **BooksController.cs** in the API project:

```csharp

[Authorize]
[ApiController]
[Route("api/[controller]")]
public class BooksController : ControllerBase
{
    private readonly LibraryService _libraryService;

    public BooksController(LibraryService libraryService)
    {
        _libraryService = libraryService;
    }

    [HttpGet]
    public IActionResult GetAllBooks()
    {
        try
        {
            var books = _libraryService.GetAllBooks();
            return Ok(books);
        }
        catch (Exception ex)
        {
            return StatusCode(500, $"Internal server error: {ex.Message}");
        }
    }

    [HttpPost]
    public IActionResult AddBook(Book book)
    {
        try
        {
            if (ModelState.IsValid)
            {
                _libraryService.AddBook(book);
                return Ok();
            }
            return BadRequest(ModelState);
        }
        catch (Exception ex)
        {
            return StatusCode(500, $"Internal server error: {ex.Message}");
        }
    }
}
```
4- Update the **LibraryService.cs** in the API project to access the user's identity:

```csharp

public class LibraryService
{
    private readonly ApplicationDbContext _context;
    private readonly IHttpContextAccessor _httpContextAccessor;

    public LibraryService(ApplicationDbContext context, IHttpContextAccessor httpContextAccessor)
    {
        _context = context;
        _httpContextAccessor = httpContextAccessor;
    }

    public IEnumerable<Book> GetAllBooks()
    {
        var userId = _httpContextAccessor.HttpContext.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        // Use the userId as needed, for example, to filter books based on user

        return _context.Books.ToList();
    }

    public void AddBook(Book book)
    {
        _context.Books.Add(book);
        _context.SaveChanges();
    }
}
```
5- Ensure that the **HttpContextAccessor** is registered in the **Startup.cs** of the API project:

```csharp

public void ConfigureServices(IServiceCollection services)
{
    // Existing code

    services.AddHttpContextAccessor();

    // Existing code
}
```
Run your IdentityServer4, Web API, and MVC projects. After logging in to the MVC app, navigate to **/Books**, and you should be able to access the Web API endpoints.

These additions include configuring JWT Bearer authentication in the API project, adding the **[Authorize]** attribute to the **BooksController**, and updating the **LibraryService** to access the user's identity in order to implement user-specific functionality in the API. Ensure that your IdentityServer4, Web API, and MVC projects are running, and the configuration is consistent across all projects.
