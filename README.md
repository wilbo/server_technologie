## server_technologie - cheatsheet voor mijn toets


### New MVC + Entity Project
  1. New Project: "ASP.NET Web Application (.NET Framework)"
  2. Create Models first!
  3. RUN IN PMC: install-Package EntityFramework


### Scaffolding DB, DBcontext & Controller
  1. Add new Controller
  2. Choose: "MVC 5 Controller with views, using Entity Framework"
      - select a Model Class
      - Create a new context (if no context already)
      - Leave name alone
  3. Run Project to fill database


### Migrations
  1. Make sure you have an existing database (only first time)
  2. RUN IN PMC: Enable-Migrations (only first time)
  3. for migration: RUN IN PMC: add-migration **migrationname**
  4. to re-add seed data: RUN IN PMC: update-database
  
  
### ONE TO MANY, tips with dropdown
1. Create Department and Employee model
```cs
// Department code
public class Department
{
	public int DepartmentID { get; set; }
	public string DepartmentName { get; set; }

	public virtual ICollection<Employee> Employees { get; set; }
}
```
```cs
public class Employee
{
	public int EmployeeID { get; set; }
	public string Name { get; set; }
	public string LastName { get; set; }

    // this id is important for the scaffolding of selectable dropdown 
	public int DepartmentID { get; set; }
	public virtual Department Department { get; set; }
}
```
2. Add controllers and views (step described above): Scaffolding DB, DBcontext & Controller
3. if neccesary (step described above): migrations
4. Add some test data
5. Profit
  
  
### MANY TO MANY, tips with checkboxes
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
// AuthorToBook code
public class AuthorToBook
{
    public int AuthorToBookID { get; set; }
    public int AuthorID { get; set; }
    public int BookID { get; set; }

    public virtual Author Author { get; set; }
    public virtual Book Book { get; set; }
}
```
2. create ViewModels
```cs
// AuthorsViewModel code
public class AuthorsViewModel
{
    public int AuthorID { get; set; }
    public string Name { get; set; }
    public string Surname { get; set; }
    public List<CheckBoxViewModel> Books { get; set; }
}
```
```cs
// CheckBoxViewModel code
public class CheckBoxViewModel
{
    public int ID { get; set; }
    public string Name { get; set; }
    public bool Checked { get; set; }
}
```
3. Add controllers and views (step described above): **Scaffolding DB, DBcontext & Controller**
4. if neccesary (step described above): **migrations**
5. Add some test data
6. Edit AuthorsController in method: "GET: Authors/Edit/5". This wil setup the model for rendering the view.
```cs
// complete code
// GET: Authors/Edit/5
public ActionResult Edit(int? id)
{
    if (id == null)
    {
        return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
    }
    Author author = db.Authors.Find(id);
    if (author == null)
    {
        return HttpNotFound();
    }

    // get all books,
    var Results = from b in db.Books
    select new
    {
        b.BookID,
        b.Title,
        // we need to know if its currently signed to the current author
        Checked = ((
            from atb in db.AuthorsToBooks
            where (atb.AuthorID == author.AuthorID) & (atb.BookID == b.BookID)
            select atb).Count() > 0)
    };

    // start creating the view model
    var MyViewModel = new AuthorsViewModel();

    // adding known values
    MyViewModel.AuthorID = id.Value;
    MyViewModel.Name = author.Name;
    MyViewModel.Surname = author.Surname;

    // constructing the checkboxes
    var MyCheckBoxList = new List<CheckBoxViewModel>();
    foreach (var item in Results)
        MyCheckBoxList.Add(new CheckBoxViewModel { ID = item.BookID, Name = item.Title, Checked = item.Checked });

    // MyCheckBoxList contains all books, some signed, some not.
    // adding the checkboxes/books. 
    MyViewModel.Books = MyCheckBoxList;
	return View(MyViewModel);
}
```
7. Edit view at ./Views/Authors/Edit.cshtml. This will create the checkboxes.
```cs
// use AuthorsViewModel
@model ManyToManyPractise.Models.AuthorsViewModel

// ... scaffold code

// add third .form-group div (around line 36)
<div class="form-group">
    @Html.LabelFor(model => model.Books, htmlAttributes: new { @class = "control-label col-md-2" })
    <div class="col-md-10">
        @for (int i = 0; i < Model.Books.Count(); i++)
        {
            @Html.EditorFor(m => Model.Books[i].Checked)
            @Html.DisplayFor(m => Model.Books[i].Name)
    
            @Html.HiddenFor(m => Model.Books[i].Name)
            @Html.HiddenFor(m => Model.Books[i].ID)
        }   
    </div>
</div>

// ... scaffold code

```
8. edit AuthorsController method: "POST: Authors/Edit/5". This will make form changes reflect in the database
```cs
// POST: Authors/Edit/5
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Edit(AuthorsViewModel author)
{
    if (ModelState.IsValid)
    {
        var MyAuthor = db.Authors.Find(author.AuthorID);

        MyAuthor.Name = author.Name;
        MyAuthor.Surname = author.Surname;

        // remove all references to books for current author
        foreach (var atb in db.AuthorsToBooks)
        {
            if (atb.AuthorID == author.AuthorID)
                db.Entry(atb).State = EntityState.Deleted;

            // just a tip:
            // what does Entry do? http://stackoverflow.com/questions/15045763/what-does-the-dbcontext-entry-do
        }

        // create a new ATB's for every book that is checked
        foreach (var book in author.Books)
        {
            if (book.Checked)
                db.AuthorsToBooks.Add(new AuthorToBook() { AuthorID = author.AuthorID, BookID = book.ID });
        }

        db.SaveChanges();
        return RedirectToAction("Index");
    }
    return View(author);
}
```
9. Run, add authors and books and edit them. In the edit form you will notice the changes in the data.
10. Profit!


### Creating a Database and connecting manualy
1. SQL Server Object Explorer
2. right-click: Databases
3. click: Create Database
4. Name database
5. in SQL Server Object Explorer, right-click database > properties to get connectionstring from infobox 
6. Edit, Copy & Paste the following snippet with a name & connectionstring above "appSettings":
```xml
  <connectionStrings>
    <add name="__NAME__" connectionString="__CONNECTIONSTRING__" providerName="System.Data.SqlClient" />
  </connectionStrings>
```

### REST API
1. Perform steps: **New MVC + Entity Project** & **Scaffolding DB, DBcontext & Controller**
2. Scaffold API Controller with: “Web API 2 Controller with actions, using Entity Framework”
3. Apply the changes to Global.asax.cs according to readme.txt that popped up:
```cs
public class MvcApplication : System.Web.HttpApplication
{
	protected void Application_Start()
	{
		// WATCH OUT: this order matters
		GlobalConfiguration.Configure(WebApiConfig.Register);
		AreaRegistration.RegisterAllAreas();
		FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
		RouteConfig.RegisterRoutes(RouteTable.Routes);
		BundleConfig.RegisterBundles(BundleTable.Bundles);
	}
}
```

### Getting REST Data
1. Create a console application
2. Add the follwoing content in Program.cs
```cs
class Program
{
	static HttpClient client = new HttpClient();

	static void Main()
	{
		RunAsync().Wait();
	}

	static async Task RunAsync()
	{
		client.BaseAddress = new Uri("http://localhost:50210/");
		client.DefaultRequestHeaders.Accept.Clear();
		client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/xml"));
			
		HttpResponseMessage response = client.GetAsync("/api/Persons").Result;
		if (response.IsSuccessStatusCode)
		{
			String xmlString = response.Content.ReadAsStringAsync().Result;
			Console.WriteLine(xmlString);
		}
	
		Console.ReadLine();
	}
}
```
3. Profit

### Session
goal: track spelers that were added this session, abondon session trough button click
1. Use session in controllers like this:

```cs
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Create([Bind(Include = "Id,Naam")] Speler speler)
{
	if (ModelState.IsValid)
	{
		db.Spelers.Add(speler);
		db.SaveChanges();

                // This is the stuff we need:
		// when a spelers id added, add its id to a list in session
		// _____________________________________________________________
                List<int> spelerIds;
                spelerIds = (List<int>)Session["speler_ids"];
                if (spelerIds == null)
                {
                    spelerIds = new List<int>();
                }
                spelerIds.Add(speler.Id);
                Session["speler_ids"] = spelerIds;
		// _____________________________________________________________

                // Opgave 3
                return RedirectToAction("Index", "home", null);
	}

	return View(speler);
}
```
2. Use session in templates like this:
```cs
@foreach (var item in Model)
{
	<tr>
	<td>
		@{
			List<int> spelerIds = (List<int>)Session["speler_ids"];
			if ((spelerIds != null) && (spelerIds.Contains(item.Id)))
			{
				<div style="color:red;">@Html.DisplayFor(modelItem => item.Naam)</div>
			}
			else
			{
				@Html.DisplayFor(modelItem => item.Naam)
			}
		}
	</td>
		
  // ... some other code
		
	</tr>
}

// end session link
@Html.ActionLink("Sessie beëindigen", "Stop", "Home", null, new { @class = "btn btn-primary", @style = "color:white" })
```
3. End session like link above and in controller like this:
```cs
public ActionResult Stop()
{
	Session.Abandon();
	return View();
}
```



