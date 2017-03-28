## server_technologie - cheatsheet voor mijn toets


### New MVC + Entity Project

  1. New Project: ASP.NET Web Application (.NET Framework)
  2. Create Models first!
  3. RUN IN PMC: install-Package EntityFramework


### Scaffolding DB, DBcontext & Controller

  1. Add new Controller
  2. Choose: MVC 5 Controller with views, using Entity Framework
      - select a Model Class
      - Create a new context
      - Leave name alone
  3. Run Project to fill database
  4. Context DBsets will generate automatically, but always check it.
  5. RUN IN PMC: Enable-Migrations


### many to many tips

1. Create Authors, Books and AuthorToBook (Tussentabel)

```cs
// Author code
public class Author
{
    public int AuthorID { get; set; }
    public string Name { get; set; }
    public string Surname { get; set; }

    // connection to tussentabel
    public virtual ICollection<AuthorToBook> AuthorsToBooks { get; set; }
}
```
```cs
// Book code
public class Book
{
    public int BookID { get; set; }
    public string Title { get; set; }

    // connection to tussentabel
    public virtual ICollection<AuthorToBook> AuthorsToBooks { get; set; }
}
```
```cs
// AuthorsToBooks code
public class AuthorToBook
{
    public int AuthorToBookID { get; set; }
    public int AuthorID { get; set; }
    public int BookID { get; set; }

    public virtual Author Author { get; set; }
    public virtual Book Book { get; set; }
}
```
2. edwedwedwed



### one to many tips

- add reference id above connection in model





// Creating a Database

1. SQL Server Object Explorer
2. RightClick: Databases
3. click: Create Database
4. Name database
5. Edit, Copy & Paste connectionstring above "<appSettings>":
  <connectionStrings>
    <add name="__NAME__" connectionString="__CONNECTIONSTRING__" providerName="System.Data.SqlClient" />
  </connectionStrings>


