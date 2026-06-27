# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an ASP.NET MVC 5 e-commerce application built on .NET Framework 4.8, integrated with the BusNet ERP system. The application handles product catalogs, shopping carts, orders, customer management, invoices, and administrative functions.

**Solution structure:**
- **NTSPJ.EShop.WebUI_CUBE** — MVC web application (main project)
- **NTSPJ.EShop.Domain** — Domain models, entities, repositories
- **NTSPJ.Eshop.BUS_EXP** — Business logic layer wrapping BusNet integrations

## Build and Run

### Building the Solution

```
msbuild NTSPJ.EShop.WebUI_CUBE.csproj /p:Configuration=Debug /p:Platform=AnyCPU
```

Or from Visual Studio:
- Open the `.sln` file in Visual Studio 2019+ (or later)
- Build > Build Solution (Ctrl+Shift+B)

The web project builds to `bin\` in the WebUI_CUBE folder.

### Running the Application

The application requires **IIS Express** for local development:
1. Open NTSPJ.EShop.WebUI_CUBE.csproj in Visual Studio
2. Press F5 or click Debug > Start Debugging
3. IIS Express will start automatically and open the default browser

**Default URL:** `http://localhost:xxxx/` (port assigned by IIS Express)

The application uses **Forms Authentication** with membership providers integrated from the BusNet database.

### Database Configuration

Database credentials are stored in `Web.config` under `appSettings`:
- `NtsPj_UtenteBusiness` — BusNet business user
- `NtsPj_PasswordBusiness` — BusNet password
- `NtsPj_ApplicationName` — ERP instance identifier

Connections are managed through the **Connections** class in the Domain layer.

## Architecture

### Layered Structure

1. **Web Layer** (NTSPJ.EShop.WebUI_CUBE)
   - Controllers organizing user actions (Articles, Cart, Orders, Account, Admin, FAQ)
   - Models/ViewModels for data binding to views
   - Areas: Admin (user/role management, reports), FAQ (knowledge base management)
   - Infrastructure: Binders, validators, custom action filters

2. **Domain Layer** (NTSPJ.EShop.Domain)
   - **Repositories**: Abstract interfaces + concrete implementations for all entities
   - **Entities**: Domain models (Cliente, Articolo, Ordine, Fattura, etc.)
   - **Services**: Cross-cutting business logic (AccountService, ClientiService, etc.)
   - **Infrastructure**: Connection management, utilities

3. **Business Logic Layer** (NTSPJ.Eshop.BUS_EXP)
   - Wrappers around BusNet COM/DLL libraries (Bn*.dll, BO*.dll)
   - Entity managers for mapping BusNet objects to domain models
   - Price/discount lookups and order processing

### Key Patterns

- **Repository Pattern**: All data access through repository interfaces (IArticoliRepository, IClientiRepository, etc.)
- **Dependency Injection**: Ninject container configured in Global.asax via `NinjectControllerFactory`
- **Model Binders**: Custom binders for INTSPJSett, IBuzEntityManager, SessioneAcquisto
- **Validation**: Custom validator attributes and providers in Infrastructure.Validators
- **Error Logging**: ELMAH for unhandled exceptions

### Data Flow

User request → Controller → Service/Repository → BusNet library → ERP database

## Code Organization

### Controllers

- `HomeController` — Portal landing page
- `ArticoliController` — Product browsing and search
- `CategorieProdottiController` — Category navigation
- `CarrelloController` — Shopping cart management
- `OrdiniController` — Order history and details
- `FattureController` — Invoice retrieval
- `ClientiController` — Customer profile management
- `AccountController` — Login/registration
- **Admin area**: UtentiController, RuoliController, ReportsController, NewsletterGestioneController
- **FAQ area**: GestFaqController, ArchiviController

### Models

- `BaseModel` — Base class with common properties
- Entity-specific models: `ArticoliListViewModel`, `CarrelloRiepilogoViewModel`, `ClienteNuovoViewModel`, etc.
- Domain entities in Entity/ folder (auto-generated from TNET code generator)

### Views

- Master pages in `Views\Shared\`
- Area-specific views in `Areas\{AreaName}\Views\`
- Partial views for reusable UI components
- Telerik controls for grids, datepickers, editors

## Key Technologies

- **UI**: ASP.NET MVC 5, Bootstrap, jQuery 3.5, Telerik Web.Mvc
- **Reporting**: Telerik Reporting
- **Advanced Controls**: DevExpress (v23.2) for admin grids and editors
- **DI Container**: Ninject 2.2
- **Editor**: Summernote for rich text editing
- **Error Handling**: ELMAH
- **ERP Integration**: BusNet (legacy COM/managed interop libraries)

## Configuration Files

- `Web.config` — Application settings, ERP credentials, database paths, ELMAH configuration
- `Global.asax` — Route registration, DI setup, error handling
- `packages.config` — NuGet package list

## Common Development Tasks

### Adding a New Repository

1. Create interface in `Domain\Abstract\INewEntityRepository.cs`
2. Implement in `Domain\Concrete\NewEntityRepository.cs`
3. Register in Ninject kernel (typically in NinjectControllerFactory)

### Adding a New Controller Action

1. Create controller or extend existing one
2. Inject required repositories/services via constructor
3. Create corresponding view/viewmodel if needed
4. Register route in `Global.asax` if a new pattern is needed

### Working with Admin Features

- Admin features are in the **Admin area** — routes: `/Admin/{Controller}/{Action}`
- Requires role-based authorization (check `RuoliController` and `UtentiController`)
- Reports use Telerik Reporting — report definitions are TRDP files in the bin folder

### Modifying the Domain Model

- Domain entities are auto-generated from the TNET code generator
- Manual extensions go in `.ext.cs` files (e.g., `Cliente.ext.cs`)
- Do not edit auto-generated `.cs` files directly

## Important Notes

- **BusNet Integration**: The application is tightly coupled to the BusNet ERP system. Most business logic is handled through BusNet business objects, not in this codebase.
- **Legacy Code**: Parts of this codebase use older patterns (TNET framework, legacy COM interop). Be cautious with refactoring in the integration layer.
- **Deployment**: The application requires IIS with ASP.NET 4.8 support and BusNet libraries installed on the server.
- **Error Logging**: Check ELMAH logs (configured in Web.config) for unhandled exceptions.
- **File Uploads**: User-uploaded files are stored in physical paths defined in Web.config (`NtsPj_CartellaFileArchivia`, `NtsPj_CartellaFileTmp`).
