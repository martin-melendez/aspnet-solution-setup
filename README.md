# ASP.NET Core MVC mit Model-First-Ansatz
> [!WARNING]
> Dieser Leitfaden bezieht sich auf die Verwendung und Implementierung von .NET 8. Alle Beispiele und Anweisungen wurden entsprechend der Spezifikationen und Funktionen von .NET 8 erstellt und getestet.

> [!IMPORTANT]
> Dieser Leitfaden zeigt die Erstellung einer ASP.NET-Anwendung mit dem Model-First-Ansatz unter Verwendung der .NET Command Line Interface (CLI). Der Model-First-Ansatz definiert Datenmodelle, um darauf aufbauend die Datenbank und den Rest der Anwendung zu generieren.
>
> Für mehr Informationen siehe:<br>
> https://learn.microsoft.com/de-at/aspnet/core/tutorials/first-mvc-app/start-mvc

## Einrichtung
Für dieses Projekt werden folgende NuGet-Pakete benötigt:
```

dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools

```

In der `appsettings.development.json` Datei muss folgender Connection String hinzugefügt werden:
```json

{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=<DATENBANK>;User=<BENUTZERNAME>;Password=<PASSWORT>TrustServerCertificate=True"
  }
}

```

## Erstellung eines User-Models
> [!TIP]
> Für diesen Leitfaden wurde eine einfache Modellklasse verwendet, um die Aufsetzung und Konfiguration zu beschleunigen, wobei für komplexere Anwendungen das Modell um zusätzliche Felder, Fremdschlüssel und weitere Elemente erweitert werden kann.

```csharp

public class User
{
    public int Id { get; set; }
    public string FirstName { get; set; } = null!;
    public string LastName { get; set; } = null!;

    etc..
}

```

## Erstellung der Context-Klasse
> [!TIP]
> Bevor die Context-Klasse erstellt wird, sollte sichergestellt werden, dass die Model-Klassen, die die Datenbanktabellen repräsentieren, bereits erstellt wurden. Die Context-Klasse verwendet diese Model-Klassen, um die Struktur der Datenbank und die Beziehungen zwischen den Tabellen zu definieren.

Hier ist ein Beispiel für eine Context-Klasse, die eine `DbSet<User>`-Eigenschaft verwendet, wobei `User` eine Model-Klasse ist, die bereits im Projekt definiert sein sollte:

```csharp

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
    {
        
    }
    
    public virtual DbSet<User> Users { get; set; }
}

```

## Konfiguration
In der Program.cs-Datei wird die Context-Klasse konfiguriert, um sie dem Dependency Injection Container hinzuzufügen. Dies ermöglicht es, den Kontext in anderen Teilen der Anwendung zu injizieren und zu verwenden.

#### Context-Klasse hinzufügen:

```csharp

builder.Services.AddDbContext<ApplicationDbContext>(options => 
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

```

Mit der Einrichtung der Context-Klasse ist die Grundlage für die Interaktion mit der Datenbank gelegt. Als nächster Schritt stehen nun die .NET CLI-Befehle im Fokus, um Migrationen zu verwalten und die Datenbankstruktur entsprechend den definierten Modellen zu aktualisieren.

## .NET Core CLI Befehle

#### Migration erstellen
```

dotnet ef migrations add <NAME>

```

#### Migration anwenden
```

dotnet ef database update

```

#### Datenbank zurücksetzen
```

dotnet ef database drop

```
