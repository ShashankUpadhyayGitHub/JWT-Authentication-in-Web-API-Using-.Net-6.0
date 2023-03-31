# JWT-Authentication-in-Web-API-Using-.Net-6.0

Introduction
In this article, we are going to create a web application using .Net 6.0 and ASP.Net Core and also implement JWT Authentication.

JWT stands for JSON Web Token digitally signed using a secret key by a token provider. It helps the resource server to verify the token data using the same secret key.

JWT consists of three parts:

Header: encoded data of the token type and the algorithm used to sign the data.
Payload: encoded data of claims intended to share.
Signature: created by signing (encoded header + encoded payload) using a secret key.
Here I am going to use Visual Studio 2022 and SQL Server 2014.

Creating Tables
First, we will create a database named “JWTAuthentication” or we can use any name and create two tables “UserInfo” and “Employee”. Open SQL Server and paste the below query to create the tables.

USE [JWTAuthentication]

CREATE TABLE [dbo].[UserInfo](
	[UserId] [int] IDENTITY(1,1) NOT NULL,
	[DisplayName] [varchar](60) NOT NULL,
	[UserName] [varchar](30) NOT NULL,
	[Email] [varchar](50) NOT NULL,
	[Password] [varchar](20) NOT NULL,
	[CreatedDate] [datetime] NOT NULL,
	CONSTRAINT [PK_UserInfo] PRIMARY KEY CLUSTERED
	(
		[UserId] ASC
	)
)
INSERT [dbo].[UserInfo] VALUES (N'Amit Mohanty', N'Admin', N'admin@abc.com', N'$admin@2022', CAST(N'2022-01-17 14:47:58.207' AS DateTime))


CREATE TABLE [dbo].[Employee](
	[EmployeeID] [int] NOT NULL,
	[NationalIDNumber] [nvarchar](15) NOT NULL,
	[EmployeeName] [nvarchar](100) NULL,
	[LoginID] [nvarchar](256) NOT NULL,
	[JobTitle] [nvarchar](50) NOT NULL,
	[BirthDate] [date] NOT NULL,
	[MaritalStatus] [nchar](1) NOT NULL,
	[Gender] [nchar](1) NOT NULL,
	[HireDate] [date] NOT NULL,
	[VacationHours] [smallint] NOT NULL,
	[SickLeaveHours] [smallint] NOT NULL,
	[rowguid] [uniqueidentifier] ROWGUIDCOL  NOT NULL,
	[ModifiedDate] [datetime] NOT NULL,
	CONSTRAINT [PK_Employee] PRIMARY KEY CLUSTERED
	(
		[EmployeeID] ASC
	)
)
INSERT [dbo].[Employee] VALUES (1, N'295847284', N'Michael Westover', N'adventure-works\ken0', N'Vice President of Sales', CAST(N'1969-01-29' AS Date), N'S', N'M', CAST(N'2009-01-14' AS Date), 99, 69, N'f01251e5-96a3-448d-981e-0f99d789110d', CAST(N'2014-06-30 00:00:00.000' AS DateTime))
INSERT [dbo].[Employee] VALUES (2, N'245797967', N'Raeann Santos', N'adventure-works\terri0', N'Vice President of Engineering', CAST(N'1971-08-01' AS Date), N'S', N'F', CAST(N'2008-01-31' AS Date), 1, 20, N'45e8f437-670d-4409-93cb-f9424a40d6ee', CAST(N'2014-06-30 00:00:00.000' AS DateTime))
INSERT [dbo].[Employee] VALUES (3, N'509647174', N'Pamela Wambsgans', N'adventure-works\roberto0', N'Engineering Manager', CAST(N'1974-11-12' AS Date), N'M', N'M', CAST(N'2007-11-11' AS Date), 2, 21, N'9bbbfb2c-efbb-4217-9ab7-f97689328841', CAST(N'2014-06-30 00:00:00.000' AS DateTime))
SQL
Create the Application
Here we will create a new project using ASP.NET Core Web API and .Net 6.0. Now open Visual Studio 2022 and follow the below steps.

Step 1



Step 2



In this step, we will select the “ASP.NET Core Web API” project type.

Step 3



Step 4



Here we will select Framework type as .NET 6.0 and also select the ASP.NET Core hosted option.

Now, our application will be created with a folder structure as given in the below image.



Install Required Nuget Packages
Go to the “Tools” menu, select NuGet Package Manager > Package Manager Console and then run the below commands to add database provider and Entity Framework Tools.

=> Install-Package Microsoft.EntityFrameworkCore
=> Install-Package Microsoft.EntityFrameworkCore.SqlServer
=> Install-Package Microsoft.AspNetCore.Authentication.JwtBearer

Adding the Model to the Application
Now we will create two Model classes that will contain the UserInfo and Employee model properties.

To do that right-click on the “JWTAuth.WebApi” project and add a New Folder as “Models”.

Then right-click on the “Models” folder and add two classes as “UserInfo.cs” and “Employee.cs”.

Now open the “UserInfo.cs” file and paste the below code to it.

namespace JWTAuth.WebApi.Models
{
    public class UserInfo
    {
        public int UserId { get; set; }
        public string? DisplayName { get; set; }
        public string? UserName { get; set; }
        public string? Email { get; set; }
        public string? Password { get; set; }
        public DateTime? CreatedDate { get; set; }
    }
}
C#
Now open the “Employee.cs” file and paste the below code to it.

namespace JWTAuth.WebApi.Models
{
    public class Employee
    {
        public int EmployeeID { get; set; }
        public string? NationalIDNumber { get; set; }
        public string? EmployeeName { get; set; }
        public string? LoginID { get; set; }
        public string? JobTitle { get; set; }
        public DateTime BirthDate { get; set; }
        public string? MaritalStatus { get; set; }
        public string? Gender { get; set; }
        public DateTime HireDate { get; set; }
        public short VacationHours { get; set; }
        public short SickLeaveHours { get; set; }
        public Guid? RowGuid { get; set; }
        public DateTime ModifiedDate { get; set; }
    }
}
C#
Adding Data Access Layer to the Application:
Now we will create a “DatabaseContext.cs” class where we define database connection. To do that right-click on the “JWTAuth.WebApi” project and add a folder as “Models”. Add the “DatabaseContext.cs” file to the “Models” folder and put the below code to it.

using Microsoft.EntityFrameworkCore;

namespace JWTAuth.WebApi.Models
{
    public partial class DatabaseContext : DbContext
    {
        public DatabaseContext()
        {
        }

        public DatabaseContext(DbContextOptions<DatabaseContext> options)
            : base(options)
        {
        }

        public virtual DbSet<Employee>? Employees { get; set; }
        public virtual DbSet<UserInfo>? UserInfos { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<UserInfo>(entity =>
            {
                entity.HasNoKey();
                entity.ToTable("UserInfo");
                entity.Property(e => e.UserId).HasColumnName("UserId");
                entity.Property(e => e.DisplayName).HasMaxLength(60).IsUnicode(false);
                entity.Property(e => e.UserName).HasMaxLength(30).IsUnicode(false);
                entity.Property(e => e.Email).HasMaxLength(50).IsUnicode(false);
                entity.Property(e => e.Password).HasMaxLength(20).IsUnicode(false);
                entity.Property(e => e.CreatedDate).IsUnicode(false);
            });

            modelBuilder.Entity<Employee>(entity =>
            {
                entity.ToTable("Employee");
                entity.Property(e => e.EmployeeID).HasColumnName("EmployeeID");
                entity.Property(e => e.NationalIDNumber).HasMaxLength(15).IsUnicode(false);
                entity.Property(e => e.EmployeeName).HasMaxLength(100).IsUnicode(false);
                entity.Property(e => e.LoginID).HasMaxLength(256).IsUnicode(false);
                entity.Property(e => e.JobTitle).HasMaxLength(50).IsUnicode(false);
                entity.Property(e => e.BirthDate).IsUnicode(false);
                entity.Property(e => e.MaritalStatus).HasMaxLength(1).IsUnicode(false);
                entity.Property(e => e.Gender).HasMaxLength(1).IsUnicode(false);
                entity.Property(e => e.HireDate).IsUnicode(false);
                entity.Property(e => e.VacationHours).IsUnicode(false);
                entity.Property(e => e.SickLeaveHours).IsUnicode(false);
                entity.Property(e => e.RowGuid).HasMaxLength(50).IsUnicode(false);
                entity.Property(e => e.ModifiedDate).IsUnicode(false);
            });

            OnModelCreatingPartial(modelBuilder);
        }

        partial void OnModelCreatingPartial(ModelBuilder modelBuilder);
    }
}
C#
Now we will create another two folders “Interface” and “Repository” to handle database-related operations.

Right-click on the “JWTAuth.WebApi” project and add two new folders as “Interface” and “Repository”.

Now add an interface to the “Interface” folder, name it as “IEmployees.cs” and put the below code to it.

using JWTAuth.WebApi.Models;

namespace JWTAuth.WebApi.Interface
{
    public interface IEmployees
    {
        public List<Employee> GetEmployeeDetails();
        public Employee GetEmployeeDetails(int id);
        public void AddEmployee(Employee employee);
        public void UpdateEmployee(Employee employee);
        public Employee DeleteEmployee(int id);
        public bool CheckEmployee(int id);
    }
}
C#
Now add a class name as “EmployeeRepository.cs” to the “Repository” folder, which will inherit “IEmployees” interface, and put the below code to it.

using JWTAuth.WebApi.Interface;
using JWTAuth.WebApi.Models;
using Microsoft.EntityFrameworkCore;

namespace JWTAuth.WebApi.Repository
{
    public class EmployeeRepository : IEmployees
    {
        readonly DatabaseContext _dbContext = new();

        public EmployeeRepository(DatabaseContext dbContext)
        {
            _dbContext = dbContext;
        }

        public List<Employee> GetEmployeeDetails()
        {
            try
            {
                return _dbContext.Employees.ToList();
            }
            catch
            {
                throw;
            }
        }

        public Employee GetEmployeeDetails(int id)
        {
            try
            {
                Employee? employee = _dbContext.Employees.Find(id);
                if (employee != null)
                {
                    return employee;
                }
                else
                {
                    throw new ArgumentNullException();
                }
            }
            catch
            {
                throw;
            }
        }

        public void AddEmployee(Employee employee)
        {
            try
            {
                _dbContext.Employees.Add(employee);
                _dbContext.SaveChanges();
            }
            catch
            {
                throw;
            }
        }

        public void UpdateEmployee(Employee employee)
        {
            try
            {
                _dbContext.Entry(employee).State = EntityState.Modified;
                _dbContext.SaveChanges();
            }
            catch
            {
                throw;
            }
        }

        public Employee DeleteEmployee(int id)
        {
            try
            {
                Employee? employee = _dbContext.Employees.Find(id);

                if (employee != null)
                {
                    _dbContext.Employees.Remove(employee);
                    _dbContext.SaveChanges();
                    return employee;
                }
                else
                {
                    throw new ArgumentNullException();
                }
            }
            catch
            {
                throw;
            }
        }

        public bool CheckEmployee(int id)
        {
            return _dbContext.Employees.Any(e => e.EmployeeID == id);
        }
    }
}
C#
Now we will add “DatabaseContext”,“IUser” and “UserManager” reference to the “Program.cs” file of the“JWTAuth.WebApi” project.

Open the “Program.cs” file and put the below code to it.

using JWTAuth.WebApi.Interface;
using JWTAuth.WebApi.Models;
using JWTAuth.WebApi.Repository;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

//Donot forgot to add ConnectionStrings as "dbConnection" to the appsetting.json file
builder.Services.AddDbContext<DatabaseContext>
    (options => options.UseSqlServer(builder.Configuration.GetConnectionString("dbConnection")));
builder.Services.AddTransient<IEmployees, EmployeeRepository>();
builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
C#
Adding the Web API Controller to the Application
Right-click on the “Controllers” folder and select “Add” then “New Item”. It will open an “Add New Item” dialog box. Select “ASP.NET” from the left panel, then select “API Controller - Empty” from templates and put the controller class name as “EmployeeController.cs”. Press Add to create the controller.

Now open the “EmployeeController.cs” file and put the below code into it.

using JWTAuth.WebApi.Interface;
using JWTAuth.WebApi.Models;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace JWTAuth.WebApi.Controllers
{
    [Route("api/employee")]
    [ApiController]
    public class EmployeeController : ControllerBase
    {
        private readonly IEmployees _IEmployee;

        public EmployeeController(IEmployees IEmployee)
        {
            _IEmployee = IEmployee;
        }

        // GET: api/employee>
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Employee>>> Get()
        {
            return await Task.FromResult(_IEmployee.GetEmployeeDetails());
        }

        // GET api/employee/5
        [HttpGet("{id}")]
        public async Task<ActionResult<Employee>> Get(int id)
        {
            var employees = await Task.FromResult(_IEmployee.GetEmployeeDetails(id));
            if (employees == null)
            {
                return NotFound();
            }
            return employees;
        }

        // POST api/employee
        [HttpPost]
        public async Task<ActionResult<Employee>> Post(Employee employee)
        {
            _IEmployee.AddEmployee(employee);
            return await Task.FromResult(employee);
        }

        // PUT api/employee/5
        [HttpPut("{id}")]
        public async Task<ActionResult<Employee>> Put(int id, Employee employee)
        {
            if (id != employee.EmployeeID)
            {
                return BadRequest();
            }
            try
            {
                _IEmployee.UpdateEmployee(employee);
            }
            catch (DbUpdateConcurrencyException)
            {
                if (!EmployeeExists(id))
                {
                    return NotFound();
                }
                else
                {
                    throw;
                }
            }
            return await Task.FromResult(employee);
        }

        // DELETE api/employee/5
        [HttpDelete("{id}")]
        public async Task<ActionResult<Employee>> Delete(int id)
        {
            var employee = _IEmployee.DeleteEmployee(id);
            return await Task.FromResult(employee);
        }

        private bool EmployeeExists(int id)
        {
            return _IEmployee.CheckEmployee(id);
        }
    }
}
C#
Run the Application and Test APIs with Postman
Before we execute the application, change the lunch URL to “api/employee” in “launchSettings.json”. When we execute the application will be able to see all employee listings like the below image.



Now we will see how to consume our service using Postman.

Postman is an API testing tool that helps developers consume and check how an API works. You can download and install Postman here.

To view the Employee list

Step 1

Open Postman and enter this endpoint: https://localhost:7113/api/employee.

Step 2 

Choose the method as GET and click Send. Now, all the employee details will be listed as shown in the below image.



To view the details of an Employee

Step 1

Open Postman and enter this endpoint: https://localhost:7113/api/employee/1.

Step 2

Choose method as GET and click Send. Now, you can see the details of the employee.



To create a new employee

Step 1

Enter this endpoint into Postman: https://localhost:7113/api/employee.

Step 2

Choose the POST method and under Body > Raw, choose type JSON and paste the employee details. By clicking Send, a new employee is created.



To update details of an employee

Step 1

Enter this endpoint into Postman: https://localhost:7113/api/employee/5.

Step 2

Choose the PUT method and under Body > Raw, choose type JSON and paste the employee details to update. By clicking on Send, the details are updated.



To delete an employee

Step 1

Enter this endpoint into Postman: https://localhost:7113/api/employee/12.

Step 2

Choose the DELETE method and click Send. Now, the employee details will be deleted from the database.



Implementation of JWT
Above we learned how to how we can consume and test our APIs in postman. But here our APIs are not secure, because anyone who knows the APIs endpoint can consume it. So to secure our APIs we will use JWT bearer token in our APIs.

Adding the Token to the Application
Right-click on the “Controllers” folder and select “Add” then “New Item”. It will open an “Add New Item” dialog box. Select “ASP.NET” from the left panel, then select “API Controller - Empty” from templates and put the controller class name as “TokenController.cs”. Press Add to create the controller.

Now open the “TokenController.cs” file and put the below code into it

using JWTAuth.WebApi.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

namespace JWTAuth.WebApi.Controllers
{
    [Route("api/token")]
    [ApiController]
    public class TokenController : ControllerBase
    {
        public IConfiguration _configuration;
        private readonly DatabaseContext _context;

        public TokenController(IConfiguration config, DatabaseContext context)
        {
            _configuration = config;
            _context = context;
        }

        [HttpPost]
        public async Task<IActionResult> Post(UserInfo _userData)
        {
            if (_userData != null && _userData.Email != null && _userData.Password != null)
            {
                var user = await GetUser(_userData.Email, _userData.Password);

                if (user != null)
                {
                    //create claims details based on the user information
                    var claims = new[] {
                        new Claim(JwtRegisteredClaimNames.Sub, _configuration["Jwt:Subject"]),
                        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
                        new Claim(JwtRegisteredClaimNames.Iat, DateTime.UtcNow.ToString()),
                        new Claim("UserId", user.UserId.ToString()),
                        new Claim("DisplayName", user.DisplayName),
                        new Claim("UserName", user.UserName),
                        new Claim("Email", user.Email)
                    };

                    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]));
                    var signIn = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
                    var token = new JwtSecurityToken(
                        _configuration["Jwt:Issuer"],
                        _configuration["Jwt:Audience"],
                        claims,
                        expires: DateTime.UtcNow.AddMinutes(10),
                        signingCredentials: signIn);

                    return Ok(new JwtSecurityTokenHandler().WriteToken(token));
                }
                else
                {
                    return BadRequest("Invalid credentials");
                }
            }
            else
            {
                return BadRequest();
            }
        }

        private async Task<UserInfo> GetUser(string email, string password)
        {
            return await _context.UserInfos.FirstOrDefaultAsync(u => u.Email == email && u.Password == password);
        }
    }
}
C#
The “TokenController” action method accepts username and password as input. It will check the user's credentials with the database to ensure the user's identity. If the username and password are valid then it will return the access token and if it’s invalid then a bad request error will be returned.

Now open “appsetting.json” and add the below code to the file

"Jwt": {
    "Key": "Yh2k7QSu4l8CZg5p6X3Pna9L0Miy4D3Bvt0JVr87UcOj69Kqw5R2Nmf4FWs03Hdx",
    "Issuer": "JWTAuthenticationServer",
    "Audience": "JWTServicePostmanClient",
    "Subject": "JWTServiceAccessToken"
  }
JavaScript
Now open “Program.cs” and add the below code to the file.

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer(options =>
{
    options.RequireHttpsMetadata = false;
    options.SaveToken = true;
    options.TokenValidationParameters = new TokenValidationParameters()
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidAudience = builder.Configuration["Jwt:Audience"],
        ValidIssuer = builder.Configuration["Jwt:Issuer"],
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
    };
});

app.UseAuthentication();
C#
In the above code, we configured authorization middleware in the startup. Here we have passed the security key when creating the token and enabled validation of Issuer and Audience. Also, we have set “SaveToken” to true, which stores the bearer token in HTTP Context. So we can use the token later in the controller.

Here is the modified “Program.cs” file code.

using JWTAuth.WebApi.Interface;
using JWTAuth.WebApi.Models;
using JWTAuth.WebApi.Repository;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

//Donot forgot to add ConnectionStrings as "dbConnection" to the appsetting.json file
builder.Services.AddDbContext<DatabaseContext>
    (options => options.UseSqlServer(builder.Configuration.GetConnectionString("dbConnection")));
builder.Services.AddTransient<IEmployees, EmployeeRepository>();
builder.Services.AddControllers();
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer(options =>
{
    options.RequireHttpsMetadata = false;
    options.SaveToken = true;
    options.TokenValidationParameters = new TokenValidationParameters()
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidAudience = builder.Configuration["Jwt:Audience"],
        ValidIssuer = builder.Configuration["Jwt:Issuer"],
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
    };
});

// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
C#
Now we add the authorization attribute to the "EmployeeController" controller, so all the APIs under this controller will be secured with the token.



Test APIs are secured by the JWT with Postman
Now when we try to get the employee list by using postman we will get a “401 Unauthorized” error.



Now, we will see how to access the APIs using the JWT token.

To create a token using Postman

Step 1

Enter this endpoint https://localhost:7113/api/token.

Step 2

Choose the POST method under Body > Raw, choose type JSON, and paste the user details. By clicking Send, user credentials will be checked, and it will generate the token.



Copy the token that was created. Under "Auth" choose type as "Bearer Token" and paste the copied token key in the "Token" field. Now by clicking on Send we will see the employee list.



In this article, we have learned how to create a REST API using .Net 6.0, ASP.NET Core, perform basic CRUD operations, create a JWT token, and secure the APIs. Hope this article will help the readers.
