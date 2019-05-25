---
layout: post
title: "Zadaci iz trece prezentacije - ASP.NET WEB API"
date: 2019-05-25
---

## Zadatak 1

### Tacka 1.3

kreirati REST endpoint koji vraća listu korisnika aplikacije  
- putanja `/project/users`

```csharp
[Route("project/users")]
[HttpGet]
public List<UserModel> GetAllUsers()
{
    return GetDB();
}
```

### Tacka 1.4

kreirati REST endpoint koji vraća korisnika po vrednosti prosleđenog ID-a
- putanja `/project/users/{id}`  
- u slučaju da ne postoji korisnik sa traženom vrednošću ID-a vratiti `null`

```csharp
[Route("project/users/{id}")]
[HttpGet]
public UserModel GetUser(int id)
{
    return GetDB().Find(x => x.id == id);
}
```

### Tacka 1.5

kreirati REST endpoint koji omogućava dodavanje novog korisnika
- putanja `/project/users`
- u okviru ove metode postavi vrednost atributa `user role` na `ROLE_CUSTOMER`
- metoda treba da vrati dodatog korisnika

```csharp
[Route("project/users")]
[HttpPost]
public UserModel PostNewUser([FromBody]UserModel user)
{
    List<UserModel> users = GetDB();
    users.Add(new UserModel
    {
        id = user.id,
        firstName = user.firstName,
        lastName = user.lastName,
        userName = user.userName,
        password = user.password,
        email = user.email,
        userRole = UserRole.ROLE_CUSTOMER
    });

    return users.Find(x => x.id == user.id);
}
```

### Tacka 1.6

kreirati REST endpoint koji omogućava izmenu postojećeg korisnika
- putanja `/project/users/{id}`
- ukoliko je prosleđen `ID` koji ne pripada nijednom korisniku metoda treba da vrati `null`, a u suprotnom vraća podatke korisnika sa izmenjenim vrednostima
- NAPOMENA: u okviru ove metode ne menjati vrednost atributa `user role` i `password`

```csharp
[Route("project/users/{id}")]
[HttpPut]
public UserModel PutChangeUserData(int id, [FromBody]UserModel user)
{
    List<UserModel> users = GetDB();
    UserModel userExisting = users.Find(x => x.id == id);

    if (userExisting != null)
    {
        userExisting.firstName = (user.firstName != null) ? user.firstName : userExisting.firstName;
        userExisting.lastName = (user.lastName != null) ? user.lastName : userExisting.lastName;
        userExisting.userName = (user.userName != null) ? user.userName : userExisting.userName;
        userExisting.email = (user.email != null) ? user.email : userExisting.email;
    }

    return users.Find(x => x.id == id);
}
```

### Tacka 1.7

kreirati REST endpoint koji omogućava izmenu atributa `userRole` postojećeg korisnika
- putanja `/project/users/change/{id}/role/{role}`
- ukoliko je prosleđen `ID` koji ne pripada nijednom korisniku metoda treba da vrati `null`, a u suprotnom vraća podatke korisnika sa izmenjenom vrednošću atributa `user role`

```csharp
[Route("project/users/change/{id}/role/{role}")]
[HttpPut]
public UserModel PutChangeUserRole(int id, UserRole role)
{
    UserModel user = GetDB().Find(x => x.id == id);
    if (user != null)
    {
        user.userRole = role;
    }

    return user;
}
```

### Tacka 1.8

kreirati REST endpoint koji omogućava izmenu vrednosti atributa `password` postojećeg korisnika
- putanja `/project/users/changePassword/{id}`
- kao parametre request-a proslediti vrednosti stare i nove lozinke
- ukoliko je prosleđen `ID` koji ne pripada nijednom korisniku metoda treba da vrati `null`, a u suprotnom vraća podatke korisnika sa izmenjenom vrednošću atributa `password`
- **NAPOMENA**: da bi vrednost atributa `password` mogla da bude zamenjena sa novom vrednošću, neophodno je da se vrednost stare lozinke korisnika poklapa sa vrednošću stare lozinke prosleđene kao parametar request-a

#### A nacin
Prosledjivanje parametara u URI-ju.

```csharp
[Route("project/users/changePassword/{id}")]
[HttpPut]
public UserModel PutChangeUserPassword(int id, [FromUri]string oldPass, [FromUri]string newPass)
{
    UserModel user = GetDB().Find(x => x.id == id);
    if (user != null)
    {
        if (user.password == oldPass)
        {
            user.password = newPass;
        }
    }

    return user;
}
```
Primer koriscenja:  
```http
PUT http://example.com/project/users/changePassword/42?oldPass=blablabla&newPass=tralala
```

#### B nacin
Prosledjivanje parametara u telu poruke, kao JSON.

```csharp
[Route("project/users/changePassword/{id}")]
[HttpPut]
public UserModel PutChangeUserPassword(int id, [FromBody]Dictionary<string, string> oldNewPass)
{
    UserModel user = GetDB().Find(x => x.id == id);
    if (user != null)
    {
        if (user.password == oldNewPass["oldPass"])
        {
            user.password = oldNewPass["newPass"];
        }
    }

    return user;
}
```

Primer koriscenja:
```http
PUT http://example.com/project/users/changePassword/42
Content-Type: application/json

{
    "oldPass": "blablabla",
    "newPass": "tralala"
}
```

### Tacka 1.9

kreirati REST endpoint koji omogućava brisanje postojećeg korisnika 
- putanja `/project/users/{id}`
- ukoliko je prosleđen `ID` koji ne pripada nijednom korisniku metoda treba da vrati `null`, a u suprotnom vraća podatke o korisniku koji je obrisan

```csharp
[Route("project/users/{id}")]
[HttpDelete]
public UserModel DeleteUser(int id)
{
    List<UserModel> users = GetDB();
    UserModel user = users.Find(x => x.id == id);
    if (user != null)
    {
        users.Remove(user);
    }

    return user;
}
```

### Tacka 1.10

kreirati REST endpoint koji vraća korisnika po vrednosti prosleđenog `username`-a
- putanja `/project/users/by-username/{username}`
- u slučaju da ne postoji korisnik sa traženim username-om vratiti `null`

```csharp
[Route("project/users/by-username/{username}")]
[HttpGet]
public UserModel GetUserByUsername(string username)
{
    // Ako je username u formatu caba.varga, metod nece raditi...
    return GetDB().Find(x => x.userName == username);
}
```

## Zadatak 2

### Tacka 2.3

kreirati REST endpoint koji vraća listu svih kategorija
- putanja `/project/categories`

```csharp
[Route("project/categories")]
[HttpGet]
public List<CategoryModel> GetCategories()
{
    return GetDB();
}
```

### Tacka 2.4

kreirati REST endpoint koji omogućava dodavanje nove kategorije
- putanja `/project/categories`
- metoda treba da vrati dodatu kategoriju

```csharp
[Route("project/categories")]
[HttpPost]
public CategoryModel PostNewCategory([FromBody]CategoryModel category)
{
    List<CategoryModel> categories = GetDB();
    categories.Add(new CategoryModel
    {
        id = category.id,
        categoryName = category.categoryName,
        categoryDescription = category.categoryDescription
    });

    return categories.Find(x => x.id == category.id);
}
```

### Tacka 2.5

kreirati REST endpoint koji omogućava izmenu postojeće kategorije
- putanja `/project/categories/{id}`
- ukoliko je prosleđen `ID` koji ne pripada nijednoj kategoriji metoda treba da vrati `null`, a u suprotnom vraća podatke kategorije sa izmenjenim vrednostima

```csharp
[Route("project/categories/{id}")]
[HttpPut]
public CategoryModel PutUpdateCategory(int id, [FromBody]CategoryModel category)
{
    List<CategoryModel> categories = GetDB();
    CategoryModel cm = categories.Find(x => x.id == id);
    if (cm != null)
    {
        if (category.categoryName != null)
        {
            cm.categoryName = category.categoryName;
        }
        if (category.categoryDescription != null)
        {
            cm.categoryDescription = category.categoryDescription;
        }
    }
    return cm;
}
```

### Tacka 2.6

kreirati REST endpoint koji omogućava brisanje postojeće kategorije
- putanja `/project/categories/{id}`
- ukoliko je prosleđen `ID` koji ne pripada nijednoj kategoriji metoda treba da vrati `null`, a u suprotnom vraća podatke o kategoriji koja je obrisana

```csharp
[Route("project/categories/{id}")]
[HttpDelete]
public CategoryModel DeleteCategory(int id)
{
    CategoryModel cm = GetDB().Find(x => x.id == id);
    if (cm != null)
    {
        GetDB().Remove(cm);
    }
    return cm;
}
```

### Tacka 2.7

kreirati REST endpoint koji vraća kategoriju po vrednosti prosleđenog ID-a
- putanja `/project/categories/{id}`
- u slučaju da ne postoji kategorija sa traženom vrednošću `ID`-a vratiti `null`

```csharp
[Route("project/categories/{id}")]
[HttpGet]
public CategoryModel GetCategory(int id)
{
    return GetDB().Find(x => x.id == id);
}
```
## Tacka 3

### Tacka 3.3

kreirati REST endpoint koja vraća listu svih ponuda
- putanja `/project/offers`

```csharp
[Route("project/offers")]
[HttpGet]
public List<OfferModel> GetOffers()
{
    return GetDB();
}
```

### Tacka 3.4

kreirati REST endpoint koji omogućava dodavanje nove ponude
- putanja `/project/offers`
- metoda treba da vrati dodatu ponudu


```csharp
[Route("project/offers")]
[HttpPost]
public OfferModel PostNewOffer([FromBody]OfferModel newOffer)
{
    List<OfferModel> offers = GetDB();
    offers.Add(newOffer);

    return offers.Find(x => x == newOffer);
}
```

### Tacka 3.5

kreirati REST endpoint koji omogućava izmenu postojeće ponude
- putanja `/project/offers/{id}`
- ukoliko je prosleđen `ID` koji ne pripada nijednoj ponudi treba da vrati `null`, a u suprotnom vraća podatke ponude sa izmenjenim vrednostima
- **NAPOMENA**: u okviru ove metode ne menjati vrednost atributa `offer status`

```csharp
[Route("project/offers/{id}")]
[HttpPut]
public OfferModel PutChangeOffer(int id, [FromBody]OfferModel offer)
{
    List<OfferModel> offers = GetDB();
    OfferModel handle = offers.Find(x => x.Id == id);
    
    if (handle != null)
    {
        handle.OfferName = offer.OfferName;
        handle.OfferDescription = offer.OfferDescription;
        handle.OfferCreated = offer.OfferCreated;
        handle.OfferExpires = offer.OfferExpires;
        handle.RegularPrice = offer.RegularPrice;
        handle.ActionPrice = offer.ActionPrice;
        handle.ImagePath = offer.ImagePath;
        handle.AvailableOffers = offer.AvailableOffers;
        handle.BoughtOffers = offer.BoughtOffers;
    }

    return handle;
}
```

### Tacka 3.6

kreirati REST endpoint koji omogućava brisanje postojeće ponude
- putanja `/project/offers/{id}`
- ukoliko je prosleđen `ID` koji ne pripada nijednoj ponudi metoda treba da vrati `null`, a u suprotnom vraća podatke o ponudi koja je obrisana

```csharp
[Route("project/offers/{id}")]
[HttpDelete]
public OfferModel DeleteOffer(int id)
{
    List<OfferModel> offers = GetDB();
    OfferModel offer = offers.Find(x => x.Id == id);
    
    if (offer != null)
    {
        offers.Remove(offer);
    }

    return offer;
}
```

### Tacka 3.7

kreirati REST endpoint koji vraća ponudu po vrednosti prosleđenog ID-a
- putanja `/project/offers/{id}`
- u slučaju da ne postoji ponuda sa traženom vrednošću `ID`-a vratiti `null`

```csharp
[Route("project/offers/{id}")]
[HttpGet]
public OfferModel GetOfferById(int id)
{
    return GetDB().Find(x => x.Id == id);
}
```

### Tacka 3.8

kreirati REST endpoint koji omogućava promenu vrednosti atributa offer status postojeće ponude
- putanja `/project/offers/changeOffer/{id}/status/{status}`
- ukoliko je prosleđen `ID` koji ne pripada nijednoj ponudi metoda treba da vrati `null`, a u suprotnom vraća podatke o ponudi koja je izmenjena

```csharp
[Route("project/offers/changeOffer/{id}/status/{status}")]
[HttpPut]
public OfferModel PutChangeOfferStatus(int id, OfferStatus status)
{
    List<OfferModel> offers = GetDB();
    OfferModel offer = offers.Find(x => x.Id == id);
    if (offer != null)
    {
        offer.OfferStatus = status;
    }

    return offer;
}
```

### Tacka 3.9

kreirati REST endpoint koji omogućava pronalazak svih ponuda čija se akcijska cena nalazi u odgovarajućem
rasponu
- putanja `/project/offers/findByPrice/{lowerPrice}/and/{upperPrice}`


```csharp
[Route("project/offers/findByPrice/{lowerPrice}/and/{upperPrice}")]
[HttpGet]
public List<OfferModel> GetOffersByActivePriceRange(double lowerPrice, double upperPrice)
{
    var offers = GetDB();
    var offersInPriceRange = offers
        .Where(x => x.ActionPrice >= lowerPrice && x.ActionPrice <= upperPrice).ToList();
    return offersInPriceRange;
}
```