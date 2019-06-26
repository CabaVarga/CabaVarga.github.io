---
layout: post
title: "Transition to services"
date: 2019-06-25
---

During the second iteration we have introduced the repository layer. Our controllers were accessing the underlying data through the Repository + UnitOfWork pattern, instead of directly working with the DbContext object.  

At the beginning of the third iteration we have made a few additional changes in order to prepare the app architecture for Dependency Injection, by removing tightly coupled associations between classes through the introduction of some key interfaces: IUnitOfWork and IGenericRepository.  

In the current state our controllers are holding a reference to IUnitOfWork. 'Business logic' processing is still happening in our endpoint methods. Our task is to isolate the business logic, remove them from the controllers and put them into a specialised, so called 'Services' layer. We need to define the interfaces for working with our key models and to move the processing code from our controllers into the corresponding methods in our service interfaces / classes.  

The proper way to explain what is happening is through an example.  

### Moving logic from UserController to UserService

REMINDER: The naming scheme has to be either of the form *User*Controller -> *User*Service OR *Users*Controller -> *Users*Service. There is no right or wrong approach here but it is important to choose one scheme and apply it throughout the solution. I'm inclined to refactor my classes into the plural form but I will probably do it the next (fourth) iteration.  

So let's start. We have a number of REST endpoint defined in the *UserController* class and we want to move the business logic to a newly created *IUserService* & *UserService* pair of classes.  

1) First step is the creation of the (empty) files, named *IUserService* and *UserService*, in the **Services** folder.  

```csharp
public interface IUserService {}
public class UserService : IUserService {}
```

Both classes will need a reference to our models:
```csharp
using Project_3rd.Models;
```

The interface has to be independent of any dependency from a data access implementation, so it will not contain anything resembling a DbContext, UnitOfWork, Repository etc...  

The implementation, on the other hand, needs to do 'real' work, therefore the associations will be included there, in our case in the form of a IUnitOfWork dependency:  
```csharp
private IUnitOfWork db;

public UserService(IUnitOfWork db)
{
    this.db = db;
}
```

Okay. The next step is the actual migration of the code from the controllers into the service. We will do it method-by-method. We need to get from this:  
```csharp
// GET: /project/users
// ZADATAK 2.1.3
[Route("project/users")]
[HttpGet]
public IEnumerable<UserModel> GetUserModels()
{
    return db.UsersRepository.Get();
}
```

... to this:  
```csharp
public IEnumerable<UserModel> GetUsers()
{
    return usersService.GetAllUsers();
}
```

A couple of observations: we are no longer accessing a *db* object held by the controller but an *usersService* object. The *db* object was an object of type UnitOfWork or IUnitOfWork (after we introduced the DI infrastructure). Our *usersService* will be of type *IUsersService* and the Dependency Injection container will have to inject the implementation into the Controller object thrugh constructor injection. To conclude: in order to our changed method to work we will need to make some changes in the controller's structure - by adding the needed field(s) and constructor. More on this later.   

NAMING: We need a name for the method we will be calling - the name has to be self-explanatory. As the method is returning all the Users naming it GetAllUsers is best practice.  

So let's add the following line of code into our interface:  
```csharp
IEnumerable<UserModel> GetAllUsers();
```

We will immediately implement the functionality in *UserService*. The standard way of doing it is by copy/pasting the code from the controller's method into the service. Our *UserService* implementation will look like this:  
```csharp
public class UserService : IUserService
{
    public IEnumerable<UserModel> GetAllUsers()
    {
        return db.UsersRepository.Get();
    }
}
```

We don't have a *db* in our *UserService* at the moment, of course. In order to compile our code, we will need to add the following code:  
```csharp
private IUnitOfWork db;

public UserController(IUnitOfWork db)
{
    this.db = db;
}
```

Ooops. So what have we done here? We were copy/pasting the above 5 lines from our *UserController* class into *UserService*. We are almost done, except that we *really* need to change  
```csharp
public UserController(IUnitOfWork db)
```

... to ....
```csharp
public UserService(IUnitOfWork db)
```

To use the service we need to add some plumbing to our controller. At the moment many methods still rely on *db* of IUnitOfWork type. If we just change in our **UserController** this part:
```csharp
private IUnitOfWork db;

public UserController(IUnitOfWork db)
{
    this.db = db;
}
```

... to this ...
```csharp
private IUserService userService;

public UserController(IUserService userService)
{
    this.userService = userService;
}
```

... the solution will compile, of course. If we want to test our transition for every method we are moving from *db* to *userService* we need to have both mechanisms at our disposal until the transition is complete.  

This is my hacky solution for this conundrum (untested so far):
```csharp
private IUserService userService;
private IUnitOfWork db;

public UserController(IUserService userService, IUnitOfWork db)
{
    this.userService = userService;
    this.db = db;
}
```

I had a bunch of problems again, after the project clean-up and bin/obj folders emptying. I had to apply **Update-Package Microsoft.CodeDom.Providers.DotNetCompilerPlatform -r** from the console. Enabling migrations and working with the database was another chore. In short, I've needed to update the connection string and the connection name - it took me a while to figure it out where my bugs were but finally it's working.  

I almost forgot: you need to update the DI functionality -> check if you have the line `UnityConfig.RegisterComponents();` in your Global.asax, add `container.RegisterType<IUserService, UserService>();` into UnityConfig.cs (for every service you'll add).  

OK. Now we have the plumbing set up (at least for the User controller and service), let's move all methods from the controller to the service. The controller methods are:

1. GetUserModel (by id)
2. PutUserModel (update the User's data)
3. PostUserModel (create a new user)
4. DeleteUserModel (delete an existing user)
5. ChangeUserRoleForUser (self-explanatory)
6. ChangePasswordForUser (same)
7. GetUserByUsername (same)

Mladen's naming scheme for the above functions:
1. GetUser
2. UpdateUser
3. CreateUser
4. DeleteUser
5. UpdateUserRole (! not 'change')
6. UpdatePassword
7. GetUserByUsername

I can either add them one-by-one or all-at-once. For a learning experience going one-by-one is preferred.  
We will start with  
```csharp
UserModel GetUser(int id);
```

Add to the interface and initiate the implementation. Change one line in the controller. Done. Super easy. 
Next. `UserModel CreateUser(UserModel user);` We have a business requirement about the UserRole field. The solution is to just set the field in the service (potentially overriding any user submitted data) and insert the user into db. **Dont forget to apply Save()**. While we are here: it is most likely expected to change the location of the UserRole enum. It has to be inside the model namespace, not inside the model class. This is a **TODO**.  