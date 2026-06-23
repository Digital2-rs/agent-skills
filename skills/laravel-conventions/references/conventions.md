# Laravel Conventions & Rules

> Internal document describing how we write Laravel projects: tech stack, folder/file structure, class naming, and conventions.
> Intended for both humans (onboarding) and AI tools (Claude Code / `CLAUDE.md` / skills).

---

## 1. Tech stack

- **Backend:** Laravel (PHP), modular monolith.
- **Frontend:** Inertia.js v3 + React + TypeScript.
- **UI:** shadcn/ui + Tailwind.
- **Tests:** Pest.

---

## 2. Folder & file structure

### 2.1 Backend (`app/`)

```
app/
├── Actions/
│   └── Addresses/
│       ├── CreateAddressAction.php
│       ├── UpdateAddressAction.php
│       └── DeleteAddressAction.php
├── Console/
│   └── Commands/
│       └── SyncRecordsCommand.php
├── DataObjects/            # DTOs
│   └── AddressData.php
├── Enums/
│   └── UserRole.php
├── Events/
│   └── PartnerRegistrationApproved.php
├── Http/
│   ├── Controllers/
│   │   ├── AddressController.php
│   │   └── SetDefaultAddressController.php
│   ├── Middleware/
│   │   └── EnsureUserIsActive.php
│   ├── Requests/
│   │   └── AddressRequest.php
│   └── Resources/
│       └── AddressResource.php
├── Jobs/
│   └── VerifyPartnerRegistration.php
├── Listeners/
│   └── SendUserInvitationNotification.php
├── Models/
│   └── Address.php
├── Notifications/
│   └── UserInvitationNotification.php
├── Policies/
│   └── AddressPolicy.php
├── Providers/
├── Queries/
│   └── ActivePartnersQuery.php
├── Services/
│   ├── Erp.php
│   ├── Nbs.php
│   └── Promobox.php
└── Support/
    ├── PriceFormatter.php
    └── PriceScale.php
```

### 2.2 Other root folders

```
bootstrap/
config/
database/
public/
resources/      # see 2.3
routes/
storage/
tests/
```

### 2.3 Frontend (`resources/`)

```
resources/
├── js/
│   ├── components/
│   │   ├── ui/          # shadcn/ui
│   │   └── reui/        # ReUI
│   ├── hooks/
│   ├── lib/
│   ├── pages/
│   │   ├── admin/
│   │   │   └── users/
│   │   │       ├── components/      # components scoped to this page
│   │   │       │   └── create-user.tsx
│   │   │       ├── index.tsx
│   │   │       └── show.tsx
│   │   └── shared/
│   │       └── dashboard.tsx
│   └── types/
└── views/
```

---

## 3. Naming conventions

### 3.1 Actions

An Action is named **verb + noun + `Action` suffix**, in `PascalCase`. Grouped by domain (`Actions/Addresses/`).

```php
// Good
CreateAddressAction
UpdateAddressAction
DeleteAddressAction
```

```php
// Bad — no suffix, no verb, or unclear scope
Address
AddressManager
HandleAddress
```

### 3.2 Controllers

A resource controller is named after the resource + `Controller` suffix. For an action outside standard CRUD, extract an **invokable** controller named after the action it performs (still with the `Controller` suffix).

```php
// Good
AddressController              // resource CRUD
SetDefaultAddressController    // invokable, single action
```

```php
// Bad
AddressesManager               // no Controller suffix
DefaultAddress                 // not clearly a controller, unclear what it does
```

### 3.3 Controller methods

Stick to the default CRUD verbs: `index`, `create`, `store`, `show`, `edit`, `update`, `destroy`. If you need a method outside this set, that's a signal you need a new (invokable) controller, not a new method. An invokable controller has a single `__invoke()`.

```php
// Good — standard CRUD on a resource controller
class AddressController
{
    public function index() { /* ... */ }
    public function store(AddressRequest $request) { /* ... */ }
    public function destroy(Address $address) { /* ... */ }
}

// Good — non-CRUD action as an invokable controller
class SetDefaultAddressController
{
    public function __invoke(Address $address) { /* ... */ }
}
```

```php
// Bad — non-standard method bolted onto a resource controller
class AddressController
{
    public function setDefault(Address $address) { /* ... */ }
    public function markVerified(Address $address) { /* ... */ }
}
```

### 3.4 Middleware

Middleware is named with a **descriptive phrase, without a `Middleware` suffix**. The name reads as an assertion about the condition being ensured.

```php
// Good
EnsureUserIsActive
EnsurePartnerIsApproved
```

```php
// Bad
UserActiveMiddleware           // redundant suffix
CheckUser                      // unclear what it ensures
ActiveMiddleware
```

### 3.5 Services

A service (wrapper around an external system/integration) is named **after the system, without a `Service` suffix**, in `Services/`.

```php
// Good
Erp
Nbs
Promobox
```

```php
// Bad
ErpService                     // redundant suffix
NbsClientService
PromoboxIntegrationService
```

### 3.6 Jobs

A job's name **describes the action it performs** (verb + object), **without a `Job` suffix**.

```php
// Good
VerifyPartnerRegistration
SyncErpPrices
PerformDatabaseCleanup
```

```php
// Bad — redundant suffix or doesn't describe the action
VerifyPartnerRegistrationJob
PartnerJob
RegistrationHandler
```

---

## 4. Writing classes

### 4.1 Action classes

An Action has **exactly one public method — `handle()`**. All logic flows through it; no additional public methods (extract helpers into protected methods or a separate Action).

Dependencies are injected via the **constructor (DI)**. `handle()` takes input (a model, DTO, primitives) and **returns a result** (model/DTO) — it does not do HTTP/redirects.

Any work that mutates data is wrapped in **`DB::transaction()`** so the operation is atomic.

```php
// Good
final class CreateAddressAction
{
    public function __construct(
        private readonly Geocoder $geocoder,
    ) {}

    public function handle(Partner $partner, AddressData $data): Address
    {
        return DB::transaction(function () use ($partner, $data) {
            $address = $partner->addresses()->create($data->toArray());

            if ($data->isDefault) {
                $this->resetOtherDefaults($partner, $address);
            }

            return $address;
        });
    }

    protected function resetOtherDefaults(Partner $partner, Address $keep): void
    {
        $partner->addresses()
            ->whereKeyNot($keep->getKey())
            ->update(['is_default' => false]);
    }
}
```

```php
// Bad — multiple public methods, no transaction, returns a redirect
class CreateAddressAction
{
    public function handle(array $data) { /* ... */ }

    public function setDefault(Address $a)        // ⟵ second public method
    {
        $a->update(['is_default' => true]);        // ⟵ multiple writes without a transaction
        return redirect()->back();                 // ⟵ an Action shouldn't do HTTP
    }
}
```

**Why:** a single entry point (`handle`) makes the Action predictable and easy to test; the transaction guarantees partial changes aren't persisted if something fails; keeping HTTP out of the Action means the logic stays usable from commands, jobs, and tests.

**Helper methods:** if you need a helper inside an Action, use `protected` (not `private`). But a protected helper is the last resort — when the logic makes sense on its own, it's **better to extract it into a separate Action class** than to keep it as a protected method. Keep it protected only when extraction doesn't make sense (e.g. a tiny step tightly coupled to this exact `handle`).

```php
// Good — standalone logic extracted into its own Action class
final class CreateAddressAction
{
    public function __construct(
        private readonly ResetDefaultAddressesAction $resetDefaults,
    ) {}

    public function handle(Partner $partner, AddressData $data): Address
    {
        return DB::transaction(function () use ($partner, $data) {
            $address = $partner->addresses()->create($data->toArray());

            if ($data->isDefault) {
                $this->resetDefaults->handle($partner, except: $address);
            }

            return $address;
        });
    }
}
```

```php
// Weaker — non-trivial logic trapped as a protected helper,
// unavailable to other Actions and harder to test in isolation
final class CreateAddressAction
{
    public function handle(Partner $partner, AddressData $data): Address { /* ... */ }

    protected function resetOtherDefaults(Partner $partner, Address $keep): void
    {
        // 15+ lines of logic that could be a standalone action
    }
}
```

### 4.2 DTOs (Data Transfer Objects)

For passing data through the application we use **Spatie Data** (`spatie/laravel-data`). A DTO class extends `Spatie\LaravelData\Data`, lives in `DataObjects/`, carries the `Data` suffix, and has **typed, promoted `public readonly` properties**.

The DTO is the primary input to an Action — the Form Request validates, then we build a DTO (`::from(...)`) and pass it to `handle()`. This way the Action never deals with raw arrays.

```php
// Good
use Spatie\LaravelData\Data;

final class AddressData extends Data
{
    public function __construct(
        public readonly string $street,
        public readonly string $city,
        public readonly string $postalCode,
        public readonly string $countryCode,
        public readonly bool $isDefault = false,
    ) {}
}

// Build from validated data and pass to the Action
$data = AddressData::from($request->validated());

$address = $createAddress->handle($partner, $data);
```

```php
// Bad — an untyped associative array is dragged through the app
$data = $request->validated();           // array<string, mixed>, no guarantees

$address = $createAddress->handle($partner, $data);

// ...or a "manual" DTO without Spatie Data: no ::from(), no type validation, no casts
class AddressData
{
    public $street;                       // no type, no readonly
    public $city;
}
```

**Why:** a typed DTO gives a clear data contract, autocompletion, and static analysis; `::from()` maps a request/model/array into an object in one step; `readonly` prevents accidental mutation as the data travels across layers.

### 4.3 Query objects

Reusable, non-trivial queries are encapsulated in a Query object. The class lives in `Queries/`, carries the `Query` suffix, and exposes a **`builder()` method that returns `Illuminate\Database\Eloquent\Builder`** — not a collection, not a result. This lets the caller decide whether to `->get()`, `->paginate()`, add another `->where()`, etc.

`builder()` accepts an optional `filters` array (defaults to empty).

```php
// Good
use Illuminate\Database\Eloquent\Builder;

final class ActivePartnersQuery
{
    /**
     * @param array<string, mixed> $filters
     */
    public function builder(array $filters = []): Builder
    {
        return Partner::query()
            ->where('is_active', true)
            ->when($filters['country_code'] ?? null, fn (Builder $q, $code) =>
                $q->where('country_code', $code)
            )
            ->when($filters['search'] ?? null, fn (Builder $q, $term) =>
                $q->where('name', 'like', "%{$term}%")
            );
    }
}

// Usage — filters are optional; the caller chooses what comes next
$partners = (new ActivePartnersQuery)->builder()->paginate(20);

$partners = (new ActivePartnersQuery)
    ->builder(['country_code' => 'RS', 'search' => 'doo'])
    ->paginate(20);
```

```php
// Bad — builder() returns an already-executed result, losing the ability
// to chain, paginate, or filter further
final class ActivePartnersQuery
{
    public function builder(): Collection           // ⟵ returns a collection, not a Builder
    {
        return Partner::query()->where('is_active', true)->get();
    }
}

// ...or the query spread directly across the controller, copied in multiple places
$partners = Partner::query()
    ->where('is_active', true)
    ->where('country_code', 'RS')
    ->paginate(20);
```

**Why:** returning a `Builder` keeps the Query object composable — a single source of truth for the query conditions, while each usage adapts it (pagination, an extra filter, `count()`) without duplicating logic.

### 4.4 Enums

We use native PHP `enum`. **Cases are written in `PascalCase`.** The enum name is `PascalCase` **without a suffix** (not `UserRoleEnum`) and lives in `Enums/`.

When the value is persisted, use a **backed enum** with explicit string values (more stable than ints when reading the database), and cast it on the model.

```php
// Good — PascalCase cases, name without a suffix
enum Suit
{
    case Clubs;
    case Diamonds;
    case Hearts;
    case Spades;
}

// Good — backed enum for persistence, explicit values
enum UserRole: string
{
    case Admin = 'admin';
    case Partner = 'partner';
    case Guest = 'guest';
}
```

```php
// Bad — redundant suffix, cases not in PascalCase
enum UserRoleEnum: string
{
    case ADMIN = 'admin';        // ⟵ UPPER_CASE
    case partner = 'partner';    // ⟵ camelCase
}
```

---

## 5. Models

### 5.1 Casts as a method

We define casts via the **`casts()` method**, not the `protected $casts` property. (Laravel 11+ supports the method; it's more readable and can use logic/expressions.)

```php
// Good
protected function casts(): array
{
    return [
        'is_active' => 'boolean',
        'role'      => UserRole::class,
        'meta'      => 'array',
        'issued_at' => 'datetime',
    ];
}
```

```php
// Bad — old property form
protected $casts = [
    'is_active' => 'boolean',
    'role'      => UserRole::class,
];
```

### 5.2 Local scope as a method + attribute

We write local scopes as a **plain method with the `#[Scope]` attribute**, without the `scope` prefix in the name. (Laravel 12+ `Illuminate\Database\Eloquent\Attributes\Scope`.)

```php
// Good
use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;

#[Scope]
protected function active(Builder $query): void
{
    $query->where('is_active', true);
}

// Usage — method name without the "scope" prefix
Partner::active()->get();
```

```php
// Bad — old "scope" prefix in the method name
public function scopeActive(Builder $query): void
{
    $query->where('is_active', true);
}
```

### 5.3 Traits

Each trait goes on its **own line with its own `use`** — not several on one line. (Produces clean diffs when a trait is added or removed.) This rule applies to all classes, not just models.

```php
// Good
class Partner extends Model
{
    use HasFactory;
    use InteractsWithMedia;
    use LogsActivity;
    use Searchable;
}
```

```php
// Bad — multiple traits on one line
class Partner extends Model
{
    use HasFactory, InteractsWithMedia, LogsActivity, Searchable;
}
```

### 5.4 No `$fillable` / `$guarded`

We use the **`nunomaduro/essentials`** package as a baseline dependency. Among other defaults (strict models, immutable dates, auto eager loading, `make:action`, opinionated Pint config) it provides **unguarded models** via its `Unguard` configurable, which we enable in `config/essentials.php`. Because models are globally unguarded, **we don't declare `$fillable` or `$guarded`**.

Mass-assignment safety is enforced at the input boundary instead — through validated Form Requests and typed DTOs (see 4.2) — not on the model. So never pass a raw `$request->all()` into `create()`/`update()`; always pass an explicit, validated payload.

```php
// config/essentials.php
return [
    NunoMaduro\Essentials\Configurables\Unguard::class => true,
    // ...
];
```

```php
// Good — no $fillable/$guarded; input is already validated upstream
class Address extends Model
{
    protected function casts(): array
    {
        return ['is_default' => 'boolean'];
    }
}

$address = $partner->addresses()->create($data->toArray()); // $data is a validated DTO
```

```php
// Bad — redundant $fillable when models are globally unguarded
class Address extends Model
{
    protected $fillable = ['street', 'city', 'postal_code', 'country_code'];
}
```

---

## 6. General rules

### 6.1 Config injection via attribute

We inject config values through the **`#[Config(...)]` attribute** on a promoted constructor property — we don't call `config()` inside the method/constructor body. (Laravel 12 `Illuminate\Container\Attributes\Config`.)

```php
// Good
use Illuminate\Container\Attributes\Config;

final class Erp
{
    public function __construct(
        #[Config('services.erp.base_url')]
        private string $baseUrl,
    ) {}
}
```

```php
// Bad — config() called manually in the constructor body
final class Erp
{
    private string $baseUrl;

    public function __construct()
    {
        $this->baseUrl = config('services.erp.base_url');
    }
}
```

**Why:** the dependency on configuration is explicit and visible in the signature; the class is easier to test (the value is easily substituted via the container) and there are no hidden `config()` calls scattered through the body.

### 6.2 Constructor property promotion

We define properties via **constructor property promotion** whenever all of them can be promoted — we don't declare them separately and assign in the body. Each argument goes on its **own line**, with a **trailing comma after the last one**.

```php
// Good
final class CreateAddressAction
{
    public function __construct(
        private Geocoder $geocoder,
        private ResetDefaultAddressesAction $resetDefaults,
    ) {}
}
```

```php
// Bad — properties declared separately and assigned manually
final class CreateAddressAction
{
    private Geocoder $geocoder;
    private ResetDefaultAddressesAction $resetDefaults;

    public function __construct(Geocoder $geocoder, ResetDefaultAddressesAction $resetDefaults)
    {
        $this->geocoder = $geocoder;
        $this->resetDefaults = $resetDefaults;
    }
}
```

### 6.3 Validation rules in array notation

Rules for a single field are written as an **array**, not as a string with the `|` separator. (Easier to add custom rule classes, and more readable with multiple rules.)

```php
// Good
public function rules(): array
{
    return [
        'email' => ['required', 'email'],
        'name'  => ['required', 'string', 'max:255'],
    ];
}
```

```php
// Bad — pipe notation
public function rules(): array
{
    return [
        'email' => 'required|email',
        'name'  => 'required|string|max:255',
    ];
}
```

### 6.4 Nullable types

When a type is nullable, use the **short notation `?Type`**, not a union with `null`.

```php
// Good
public ?string $variable;
```

```php
// Bad
public string | null $variable;
```

### 6.5 Void return type

If a method returns nothing, mark the return type explicitly with **`void`**. It makes the intent clearer to anyone using the code.

```php
// Good
public function scopeArchived(Builder $query): void
{
    $query->where('archived', true);
}
```

```php
// Bad — no return type
public function scopeArchived(Builder $query)
{
    $query->where('archived', true);
}
```

### 6.6 If statements

**Always use curly braces**, even for a single-line block.

```php
// Good
if ($condition) {
    // ...
}
```

```php
// Bad
if ($condition) // ...
```

**Happy path last.** The unhappy path (errors, early returns) comes first; the happy path stays at the end, unindented — more readable.

```php
// Good
if (! $goodCondition) {
    throw new Exception;
}

// happy path
```

```php
// Bad
if ($goodCondition) {
    // happy path
}

throw new Exception;
```

**Avoid `else`.** Most of the time it can be refactored with an early return, which also pushes the happy path to the end.

```php
// Good
if (! $conditionA) {
    return;
}

if (! $conditionB) {
    return;
}

// A and B passed
```

```php
// Bad
if ($conditionA) {
    if ($conditionB) {
        // A and B passed
    } else {
        // ...
    }
} else {
    // ...
}
```

**Separate `if` statements over a compound condition.** Easier to debug.

```php
// Good
if (! $conditionA) {
    return;
}

if (! $conditionB) {
    return;
}

if (! $conditionC) {
    return;
}

// do stuff
```

```php
// Bad
if ($conditionA && $conditionB && $conditionC) {
    // do stuff
}
```

### 6.7 Whitespace

Statements should be allowed to breathe. Generally add a blank line between statements, unless they're a sequence of single-line equivalent operations. This isn't strictly enforceable — it's a matter of what looks best in context. No blank lines immediately after `{` or before `}`.

```php
// Good — statements separated by blank lines
public function getPage($url)
{
    $page = $this->pages()->where('slug', $url)->first();

    if (! $page) {
        return null;
    }

    if ($page['private'] && ! Auth::check()) {
        return null;
    }

    return $page;
}

// Good — a sequence of equivalent operations stays together
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamps();
});
```

```php
// Bad — everything cramped together
public function getPage($url)
{
    $page = $this->pages()->where('slug', $url)->first();
    if (! $page) {
        return null;
    }
    if ($page['private'] && ! Auth::check()) {
        return null;
    }
    return $page;
}

// Bad — blank line right after {
if ($foo) {

    $this->foo = $foo;
}
```

---

## 7. Migrations

### 7.1 No `down()` method

Migrations are **forward-only** — they contain only an `up()` method, no `down()`. We don't rely on rollbacks; to reverse a change, write a new migration. This keeps migrations simpler and avoids maintaining reverse logic that's rarely correct in practice.

```php
// Good — only up()
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('addresses', function (Blueprint $table) {
            $table->id();
            $table->foreignId('partner_id')->index();
            $table->string('street');
            $table->string('city');
            $table->timestamps();
        });
    }
};
```

```php
// Bad — includes a down() method
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('addresses', function (Blueprint $table) {
            // ...
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('addresses');
    }
};
```

### 6.8 JSON resources without wrapping

We disable the default `data` wrapper on API resources by calling **`JsonResource::withoutWrapping()`** once, in a service provider's `boot()`. Resource responses are then returned flat, without a top-level `data` key.

```php
// Good — configured once in AppServiceProvider::boot()
use Illuminate\Http\Resources\Json\JsonResource;

public function boot(): void
{
    JsonResource::withoutWrapping();
}

// Response shape:
// { "id": 1, "street": "...", "city": "..." }
```

```php
// Bad — leaving the default wrapping in place
// { "data": { "id": 1, "street": "...", "city": "..." } }
```

---

## 8. For the AI agent: Laravel Boost

This project has **Laravel Boost** installed (a dev-dependency MCP server by the Laravel team). When you are an AI coding agent working in this repo, use Boost's tools instead of guessing — they reflect the real state of the app and the exact installed package versions.

**Before writing or changing code, use `search-docs`.** It returns version-specific documentation for the packages actually installed here, so you use the correct API for our Laravel/Inertia/etc. versions rather than relying on stale training data.

- Use multiple broad, topic-based queries in one call, e.g. `['rate limiting', 'routing rate limiting', 'middleware']`. Most relevant results come first.
- Do **not** put package names in the query — installed package versions are passed to the API automatically. Use `test resource table`, not `filament 4 test resource table`.

**Use the right tool for the job:**

- `database-schema` — inspect table structure (columns, types, indexes) before adding columns or writing migrations/queries.
- `database-query` — read-only queries when you only need to read data (prefer this over `tinker` for reads).
- `tinker` — execute PHP in the app context: inspect Eloquent models, create fixtures, debug.
- `list-routes`, `list-artisan-commands`, `get-config` — inspect routes, available commands, and config values.
- `read-log-entries`, `browser-logs`, `last-error` — diagnose backend/frontend errors (e.g. triage a 500 or a white screen) without leaving the editor.
- `application-info` — PHP/Laravel versions, ecosystem packages, and models.

**Don't hallucinate APIs.** If you're unsure how something works in our versions, call `search-docs` first; verify schema with `database-schema` and behavior with `tinker` before proposing a change.

**Project guidelines:** repo-specific AI guidelines live in `.ai/guidelines/*.blade.php` and are compiled into the agent files (`CLAUDE.md` / `AGENTS.md`) by `boost:install` / `boost:update`. This conventions document is the human-and-AI source of truth and complements those generated guidelines — when they disagree, this document wins and should be updated to match.