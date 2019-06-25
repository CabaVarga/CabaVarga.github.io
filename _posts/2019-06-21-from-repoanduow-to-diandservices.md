---
layout: post
title: "Getting from repositories and Unit of Work to Dependency Injection and services"
date: 2019-06-21
---

GIVEN: Controllers that work with an UnitOfWork as a mediator and Repositories accessible through the UOW object.

TASK: Get the project ready for applying Dependency Injection.

SOLUTION: Create interfaces for the UnitOfWork and (Generic)Repository classes. Update the (concrete) classes to implement the interfaces. Change the controller(s) to work with interfaces, not their implementation. Install a DI library (container) that will instantiate the needed objects (types). Config the library to work with your project.

#### Install a DI library (Unity)

The first step is very simple: Right click on project, "Manage NuGet Packages", Browse tab, write `unity.webapi` in the search bar, install the package.

Next step: add the line ...
```csharp
UnityConfig.RegisterComponents();
```

... to the `Application_Start()` method in `Global.asax`.

Finally, you need to register the types. Type registration is handled in the `UnityConfig.cs` file inside the `App_Start` folder. You need to register every Dependency Injection point you have in your project. These are currently the `IUnitOfWork` and `IGenericRepository` interfaces (you need to provide their implementations - the IUnitOfWork for the controller(s) and the IGenericRepository for the property injection inside UnitOfWork.cs) and the `DbContext` implementation for the `UnitOfWork` constructor. FOR EVERY ENTITY you need to register the associated IGenericRepository!

```csharp
container.RegisterType<IUnitOfWork, UnitOfWork>();
container.RegisterType<IGenericRepository<UserModel>, GenericRepository<UserModel>>();
```

etc... For the `DbContext` you need an additional parameter:
```csharp
container.RegisterType<DbContext, DataAccessContext>(new HierarchicalLifetimeManager());
```

After you introduce the services you will need to register them also.


#### Create interfaces for the UnitOfWork and GenericRepository classes

We have two files in the Repositories folder, *GenericRepository.cs* and *UnitOfWork.cs*. We need to create two new files, *IGenericRepository.cs* and *IUnitOfWork.cs*.  

##### IGenericRepository

The first line will be 
```csharp
public interface IGenericRepository<TEntity> where TEntity : class
```

In the interface's body we need to add the existing methods with the same signature. We have 6 methods: `GetByID`, `Insert`, `Delete`, `Get`, `Delete` (an overload) and `Update`.

Insert the following code:
```csharp
TEntity GetByID(object id);

void Insert(TEntity entity);

void Delete(object id);

IEnumerable<TEntity> Get(
    Expression<Func<TEntity, bool>> filter = null,
    Func<IQueryable<TEntity>, IOrderedQueryable<TEntity>> orderBy = null,
    string includeProperties = "");

void Delete(TEntity entityToDelete);

void Update(TEntity entityToUpdate);
```

To connect the `GenericRepository` with the newly added interface we need to change the class headline:
```csharp
public class GenericRepository<TEntity> : IGenericRepository<TEntity> where TEntity : class
```

And that's it.

##### IUnitOfWork

First line:
```csharp
public interface IUnitOfWork
```

The UnitOfWork has one central method, the `Save()` method. In addition we need to add all the repositories that we will work with. In our case the body will look like this:
```csharp
IGenericRepository<UserModel> UsersRepository { get; }

IGenericRepository<CategoryModel> CategoriesRepository { get; }

IGenericRepository<OfferModel> OffersRepository { get; }

IGenericRepository<BillModel> BillsRepository { get; }

IGenericRepository<VoucherModel> VouchersRepository { get; }

void Save();
```

The `IDisposable` interface was not added to the IUnitOfWork probably so that we can create implementations without using any disposable resources.

We have to update the `UnitOfWork.cs` file in order to connect our class with the interface. AT THIS MOMENT THE NEED FOR HAVING A DI CONTAINER INSTALLED ARISES. Our code will look like this:

```csharp
public class UnitOfWork : IUnitOfWork, IDisposable
{
    private DbContext context;

    public UnitOfWork(DbContext context)
    {
        this.context = context;
    }

    [Dependency]
    public IGenericRepository<UserModel> UsersRepository { get; set; }

    [Dependency]
    public IGenericRepository<CategoryModel> CategoriesRepository { get; set; }

    [Dependency]
    public IGenericRepository<OfferModel> OffersRepository { get; set; }

    [Dependency]
    public IGenericRepository<BillModel> BillsRepository { get; set; }

    [Dependency]
    public IGenericRepository<VoucherModel> VouchersRepository { get; set; }
```

And this will not work without DI already in place.

Let just paste the `Dispose` logic here, for reference:
```csharp
private bool disposed = false;
protected virtual void Dispose(bool disposing)
{
    if (!this.disposed)
    {
        if (disposing)
        {
            context.Dispose();
        }
    }
    this.disposed = true;
}

public void Dispose()
{
    Dispose(true);
    GC.SuppressFinalize(this);
}
```

#### Update the controllers to work with IUnitOfWork

By changing UnitOfWork to have a constructor ready for DI our controllers won't compile until we make some changes. The hacky way would be to just add an empty constructor to UnitOfWork. The project would compile but it will not work, because UnitOfWork will not work without the DbContext.  

We need to implement the following in our controllers

Delete the dependency:
```csharp
private UnitOfWork db = new UnitOfWork();
```

My solution is not working because currently my repos are named `UserRepository`, `BillRepository` etc, while Mladen's implementation is using plural naming, `UsersRepository` etc. I can either change the interface to my naming or change my naming to plural. I really think the plural naming scheme is better so I will do the latter.  


