---
layout: post
title: "ASP.NET WEB API 2 Uvod - Primer 4"
date: 2019-05-19
---

```csharp
[Route("api/BankClientRest/{clientId}")]
[HttpPut]
// PUT: api/BankClientRest/5
public BankClientModel Put(int clientId, [FromBody]BankClientModel client)
{
    var clients = new List<BankClientModel>
    {
        new BankClientModel(1, "Mladen", "Kanostrevac", "mk@gmail.com"),
        new BankClientModel(2, "Ivan", "Vasiljevic", "iv@gmail.com"),
        new BankClientModel(3, "Caba", "Varga", "cv@gmail.com")
    };

    Debug.WriteLine($"Id: {client.Id}\nName: {client.Name}\nSurname: {client.Surname}\nEmail: {client.Email}");

    BankClientModel existingClient = clients.Find(x => x.Id == clientId);
    if (existingClient != null)
    {
        existingClient.Name = client.Name;
        existingClient.Surname = client.Surname;
        existingClient.Email = client.Email;
        return existingClient;
    }
    else
    {
        return null;
    }
            
}
```
Sta radimo, korak-po-korak?

Primer pocinje ovako: "Kreirati REST endpoint koji omogucuje izmenu postojeceg klijenta. Putanja `/api/BankClientRest/{clientId}`"

Na osnovu navedenog znamo da treba da napravimo novi metod u postojecoj klasi, koji ce obradjivati zahteve koji ciljaju definisanu putanju. Prvi korak je definisanje da metod koji sledi obradjuje takve zahteve, a to radimo pisanjem atributa `Route` sa argumentom putanje, dakle `api/BankClientRest/{clientId}`. Znak `/` na pocetku izostavljamo! Dakle prvi red izgleda ovako:
```csharp
[Route("api/BankClientRest/{clientId}")]
```
Tekst primera dalje sledi: "Ukoliko je prosledjen id sa vrednoscu 1 metoda vraca podatke o klijentu sa izmenjenim vrednostima. U suprotnom vraca `null` vrednost."

Za nastavak nam treba i poslednja linija zadatka: "ime metoda mora da pocinje sa Put ili metoda ima atribut HttpPut".
Sada konacno znamo koju vrstu HTTP naredbi obradjuje metoda: to su PUT naredbe. U nasem primeru smo se odlucili da to eksplicitno naglasimo, pisanjem odgovarajuceg atributa iznad pocetka definicije metode:
```csharp
[HttpPut]
```
Sledi definicija zaglavlja metode. Zaglavlje sadrzi sledece: vidljivost (public, private, protected itd), tip koji se vraca (void, int, string, itd, u nasem slucaju BankClientModel), naziv metode i parametri metode, unutar zagrada (ulazni podaci za metod).
To smo mi ovako definisali:
```csharp
public BankClientModel Put(int clientId, [FromBody]BankClientModel client)
```
Treba obratiti paznju na parametre: `int clientID` se poklapa sa tipom i nazivom iz rute! Da smo u ruti napisali `[Route("api\BankClientRest\{idKlijenta}")]` onda bismo i u parametrima morali imenovati parametar kao `idKlijenta`.

Drugi parametar pocinje atributom `[FromBody]`. Taj atribut oznacava da ulazni parametar prima telo HTTP poruke. Telo poruke treba pretvoriti u tip `BankClientModel` i imenovati sa `client`. To pretvaranje se vrsi u pozadini.

Sledi definicija samog metoda.
Tu se resenja sa slajda i ono sto smo radili na casu razilaze. Ja sam zadatak prosirio da radi za svaki clientId ako postoji klijent sa tim identifikacionim brojem. Elem, da se drzimo teksta zadatka.

Sta nam treba? Vratimo se na tekst zadatka: "Ukoliko je prosledjen id sa vrednoscu 1 metoda vraca podatke o klijentu sa *izmenjenim vrednostima*. U suprotnom vraca `null` vrednost." Obrati paznju na *sa izmenjenim vrednostima*. Naime, PUT naredba HTTP-a upravo sluzi za menjanje nekih podataka, slicno UPDATE naredbi kod SQL-a.

Dakle, programska logika izgleda ovako:
```
IF (ID = 1)
  IZMENI PODATKE
  VRATI IZMENJENE PODATKE
ELSE
  VRATI VREDNOST 'NULL'
```
Isto to, prevedeno na `C#` i nas konkretni primer (podatak nije proizvoljan podatak vec objekat tipa `BankClientModel`) izgleda ovako, opet korak-po-korak:
```csharp
BankClientModel bcb = new BankClientModel(1, "Ivan", "Vasiljevic", "ivan.vasiljevic@hotmail.com");
```
Ova linija koda sluzi samo da napravimo objekat tipa BankClientModel kojem mozemo pristupiti.
```csharp
if (clientId == 1) {
  bcb.Name = client.Name;   // client.Name povlacimo iz ulaznog parametra client
  return bcb;
}
else {
  return null;
}
```

I to je to.

Sada da objasnim sta sam ja radio. Kao prvo, recimo da smo zadatak izmenili ovako: "Ukoliko postoji klijent za prosledjenu id vrednost metoda vraca podatke o klijentu sa *izmenjenim vrednostima*. U suprotnom vraca `null` vrednost".

Da bismo resili taj zadatak moramo uraditi sledece:
```
PRONADJI KLIJENTA SA (KLIJENT.ID = ID)
IF (PRONADJEN KLIJENT)
  IZMENI PODATKE
  VRATI IZMENJENE PODATKE
ELSE
  VRATI VREDNOST 'NULL'
```

Kada jednom znas kojim koracima treba da dodjes do resenja implementacija je samo stvar uvezbanosti.
