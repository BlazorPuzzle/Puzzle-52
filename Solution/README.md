# Blazor Puzzle #52

## Who?

YouTube Video: https://youtu.be/o7XMC6ckWBc

Blazor Puzzle Home Page: https://blazorpuzzle.com

### The Challenge:

This is a simple Blazor Web App with Global Server Interactivity and basic authentication.

When the user registers, we want to capture their first and last names, even though these fields are not part of the ASP.NET Core Identity default data schema.

How can we do this?

### The Solution:

There are a couple solutions to this problem. One is to add the FirstName and LastName fields as claims. We didn't prefer this solution because Claims is just a list of name-value pairs. It works, but we think there's a better solution, which came in detail from Blazor Puzzle regular, Bill Scharf. 

Step 1 is to modify the *ApplicationUser.cs* file to support the new fields:

```c#
using Microsoft.AspNetCore.Identity;
using System.ComponentModel.DataAnnotations;

namespace Puzzle_52.Data
{
    // Add profile data for application users by adding properties to the ApplicationUser class
    public class ApplicationUser : IdentityUser
    {
        public ApplicationUser()
        {
        }

        [Required]
        public string FirstName { get; set; } = string.Empty;

        [Required]
        public string LastName { get; set; } = string.Empty;
    }
}
```

Step 2 is to add a new migration in the **Package Manager Console**:

```
PM> add-migration "Added FirstName and LastName to ApplicationUser"
```

The next few steps are to modify *Components\Account\Pages\Register.razor*

First, add the new fields to the `InputModel` class defined at the bottom of the page:

```c#
private sealed class InputModel
{
    [Required]
    [EmailAddress]
    [Display(Name = "Email")]
    public string Email { get; set; } = "";

    [Required]
    [StringLength(100, ErrorMessage = "The {0} must be at least {2} and at max {1} characters long.", MinimumLength = 6)]
    [DataType(DataType.Password)]
    [Display(Name = "Password")]
    public string Password { get; set; } = "";

    [DataType(DataType.Password)]
    [Display(Name = "Confirm password")]
    [Compare("Password", ErrorMessage = "The password and confirmation password do not match.")]
    public string ConfirmPassword { get; set; } = "";

    [Required]
    [Display(Name = "First Name")]
    public string FirstName { get; set; } = "";

    [Required]
    [Display(Name = "Last Name")]
    public string LastName { get; set; } = "";
}
```

Next, add the fields to the `<EditForm>`:

```xml
<div class="form-floating mb-3">
    <InputText @bind-Value="Input.FirstName" class="form-control" aria-required="true" />
    <label for="firstname">First Name</label>
    <ValidationMessage For="() => Input.FirstName" class="text-danger" />
</div>
<div class="form-floating mb-3">
    <InputText @bind-Value="Input.LastName" class="form-control" aria-required="true" />
    <label for="lastname">Last Name</label>
    <ValidationMessage For="() => Input.LastName" class="text-danger" />
</div>
```

Finally, add the values to the `user` variable in the `RegisterUser` method:

```c#
public async Task RegisterUser(EditContext editContext)
{
    var user = CreateUser();

    user.FirstName = Input.FirstName;
    user.LastName = Input.LastName; 
    
    ...
```

Now, any time you want to access the `FirstName` and `LastName` properties, just examine the `ApplicationUser` object, which is accessed in the usual way.

Boom!