# Live-Project-Code-Summary

In this project I was responsible for creating a Blog Authors section for a Theater Website, including styling the page. Additionally, I styled the home page for the Website. To accomplish this, I used MVC with JQuery and Bootstrap 4. The Blog Authors section used an MVC model to retrieve information from a database. The Blog Author Model has the following properties: Id, Name, Bio, Date Joined and Date Left (optional)
```csharp
using System;
namespace TheatreCMS3.Areas.Blog.Models
{
    public class BlogAuthor
    {
        public int BlogAuthorId { get; set; }
        public string Name { get; set; }
        public string Bio { get; set; }
        public DateTime DateJoined { get; set; }
        public DateTime? DateLeft { get; set; }
    }
}
```
From the home page, the Blog Authors section could be accessed through a navbar.
Here's an example screenshot:

![Screenshot 2023-05-16 161712](https://github.com/jordanritzie19/Live-Project-Code-Summary/assets/118029206/89789f33-75e5-4b3c-a608-81726ba92dd1)


I created the CRUD pages for the Blog Authors, along with an Access Denied page. Here is what the Create and Edit Pages look like: 

![image](https://github.com/jordanritzie19/Live-Project-Code-Summary/assets/118029206/711c37c4-62b8-4e57-9853-7ce8f503f059)

With the necessary adjustments to the Edit Page for it to make sense. Essentially, I centered a form element, where the user can input the data needed to create (or update) the Model. First Name, Blog and Date Joined fields being mandatory in order to update the database. You could also choose to go back to the home page quickly by  clicking the 'Back to List.' button.

Here's the Razor syntax code for the buttons, utilizing Bootstrap 4:
```razor
<div class="form-group">
  <div class="col-md-offset-2 col-md-10">
      <button type="button" class="btn btn-dark">@Html.ActionLink("Back to List", "Index", null, new { @class = "text-white"})</button>
      <input type="submit" value="Save" class="btn btn-success" />
  </div>
</div>
```

Similarly, the details and delete pages had these layouts:

![image](https://github.com/jordanritzie19/Live-Project-Code-Summary/assets/118029206/91a96a3e-eddc-471a-a81b-a64634ca068f)
![image](https://github.com/jordanritzie19/Live-Project-Code-Summary/assets/118029206/2b559315-c8f0-44b7-beed-e9a453b37fb5)

The Edit button took users to the respective Edit page, while the Delete button redirected them to the Delete page. Upon confirming the deletion, the model was permanently removed from the database and no longer appeared on the website. However, these functionalities were replaced with a more dynamic approach in the index page.

I created a partial view to be used on the home page, representing each instance of a Blog Author. This partial view displayed a card similar to the Edit page. Here's an example of two populated Blog Author cards using the partial view:

![image](https://github.com/jordanritzie19/Live-Project-Code-Summary/assets/118029206/81ea349a-22c5-4dde-af3a-9c27fa6b4c38)

The code for the partial view essentially being the below, with the buttons also using the BlogAuthorId to properly target the correct Author to change.
```razor
@model IEnumerable<TheatreCMS3.Areas.Blog.Models.BlogAuthor>

@foreach (var item in Model)
{
    if (item.DateLeft == null)
    {                                              
        <div class="mt-5 blogAuthor-details--authorContainer" data-id="@item.BlogAuthorId"> @*this stores the model id to be accessed later*@
            html...
        </div>
```

The code for the partial view loops through each item in the database, and displays only the Authors that have not left yet. The delete button setting the date left, instead of removing the Author from the database altogether.

When you push the Delete button, marked by the 'trash' symbol from fontawesome, a confirmation modal pops up. This modal does not come from the partial views and regardless of which BlogAuthor you try to delete, the same Modal pops up. There is only one modal, and we use Javascript to pull the data-id from the triggering button to know which BlogAuthor to delete. This id gets stored in a hidden input in the modal. Here's what the modal looks like:

![image](https://github.com/jordanritzie19/Live-Project-Code-Summary/assets/118029206/f59be5b1-2079-4ec9-8a02-cf807f602d79)

To hide this modal the users could either press somewhere on the page outside the modal, or the 'Close' button.

Here's the JavaScript code used to extract the BlogAuthorId, set the Date Left property, and fade out the card without neccesitating a reload:
```javascript
// triggers on modal show
    $('#blogAuthor-index--Modal').on('show.bs.modal', function (event) {
        var id = $(event.relatedTarget).data('id');  //takes the data-id of the triggering button
        $('#blogAuthor-index--id').val(id);      //this value is then set to the hidden input in modal
    });

    function setDateLeft() {
        var id = $('#blogAuthor-index--id').val(); //gets the hidden input
        $.ajax({
            url: '@Url.Action("setDateLeft", "BlogAuthors")',
            type: 'POST',
            data: { id: id },     //sends id as string to controller
            // runs when function succeeds
            success: function (data) {
                var blogAuthor = $('.blogAuthor-details--authorContainer[data-id="' + id + '"]');

                //animations + removal
                blogAuthor.fadeOut(750, function () {   //time is in milliseconds
                    blogAuthor.remove();
                });
            }
        });
    };
```

And in the controller for setDateLeft:
```csharp
public ActionResult setDateLeft(string id) //string id is received
      {
          BlogAuthor blogAuthor = db.BlogAuthors.Find(Convert.ToInt32(id)); //converts string to int, and finds the matching entity
          blogAuthor.DateLeft = DateTime.Now; 
          db.SaveChanges();
          return RedirectToAction("Index");
      }
```

In hindsight, storing the BlogAuthorId as a string in my partial view was needless, and should've stayed an integer. This would simplify the code, and allow me to no longer need to convert the sent Id to an integer within my controller. Regardless, it is functional, and I was encouraged to learn from this, and move on to the next story.

Lastly, I created a new Application User 'Head Author' that has access to the edit and delete functions, and seeded it to the page. In order to be the HeadAuthor, you would have to log in to the page with the HeadAuthor credentials. The model looks like this:

```csharp
using Microsoft.AspNet.Identity;
using Microsoft.AspNet.Identity.EntityFramework;
using TheatreCMS3.Models;

namespace TheatreCMS3.Areas.Blog.Models
{
    public class HeadAuthor : ApplicationUser
    {
        public int ViewsPerMonth { get; set; }
        public int AuthorsHired { get; set; }
        public int AuthorsLetGo { get; set; }

        public static void Seed()
        {            
            var roleManager = new RoleManager<IdentityRole>(new RoleStore<IdentityRole>(new ApplicationDbContext()));
            if (!roleManager.RoleExists("Head Author"))
            {
                var role = new IdentityRole { Name = "Head Author" };
                roleManager.Create(role);
            }

            var userManager = new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(new ApplicationDbContext()));

            var headauthor = new HeadAuthor {
                ViewsPerMonth = 0,
                AuthorsHired = 0,
                AuthorsLetGo = 0,
                Email = "headauthor@admin.com",
                EmailConfirmed = true,
                PhoneNumber = "5131234567",
                PhoneNumberConfirmed = true,
                TwoFactorEnabled = false,
                LockoutEnabled = false,
                AccessFailedCount = 0,
                UserName = "headauthor",
            };

            userManager.Create(headauthor, "headauthorpassword");
            userManager.AddToRole(headauthor.Id, "Head Author");
        }
    }    
}
```

This HeadAuthor model gets added to the table with a 'Table-per-Hierarchy' structure. 
