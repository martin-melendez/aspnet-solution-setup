# ASP.NET Core MVC mit Model-First-Ansatz
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

> [!WARNING]
> Connection Strings sollten niemals direkt in `appsettings` oder im Code gespeichert werden. Es wird empfohlen, Umgebungsvariablen oder sichere Speicher wie Azure Key Vault zu verwenden, um Datenbankverbindungen zu schützen.

## Erstellung eines User-Models
> [!TIP]
> Für diesen Leitfaden wurde eine einfache Modellklasse verwendet, um die Aufsetzung und Konfiguration zu beschleunigen, wobei für komplexere Anwendungen das Modell um zusätzliche Felder, Fremdschlüssel und weitere Elemente erweitert werden kann.

```csharp

public class User
{
    public int Id { get; set; }
    public string FirstName { get; set; } = null!;
    public string LastName { get; set; } = null!;
}

```

## Beziehungen definieren
In diesem Abschnitt wird ein Beispiel für eine 1-zu-n-Beziehung (One-to-Many) zwischen zwei Entitäten `Role` und `User` gezeigt. Eine `Role` kann mehreren `User` Objekten zugeordnet sein, während ein `User` nur eine `Role` haben kann.

#### `Role` Klasse
```csharp

public class Role
{
    public int Id { get; set; }
    public string RoleName { get; set; } = null!;

    public virtual ICollection<User>? Users { get; set; }
}

```

#### `User` Klasse
```csharp

public class User
{
    public int Id { get; set; }
    public string FirstName { get; set; } = null!;
    public string LastName { get; set; } = null!;

    public int? RoleId { get; set; }
    public virtual Role? Role { get; set; }
}

```

> [!TIP]
> Das Schlüsselwort `virtual` wird bei den Eigenschaften `Users` und `Role` verwendet. Dies ermöglicht es, dass diese Beziehungen bei Bedarf automatisch aus der Datenbank geladen werden, ohne dass sie immer sofort beim Laden des Objekts dabei sein müssen. Das hilft, die App schneller zu machen, weil sie nur die Infos lädt, die gerade benötigt werden.

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
