# PHP 8 - Nowości i Funkcje

## Spis treści
- [Atrybuty (Attributes)](#atrybuty-attributes)
- [Union Types](#union-types)
- [Match Expressions](#match-expressions)
- [Named Arguments](#named-arguments)
- [Enumeracje (Enums)](#enumeracje-enums)
- [Constructor Property Promotion](#constructor-property-promotion)
- [Nullsafe Operator](#nullsafe-operator)
- [Mixed Type](#mixed-type)
- [Podsumowanie](#podsumowanie)

---

## Atrybuty (Attributes)

Atrybuty to nowoczesna alternatywa dla komentarzy DocBlock, pozwalająca na dodawanie metadanych bezpośrednio do kodu.

### Składnia podstawowa
```php
#[Route('/users', methods: ['GET'])]
class UserController
{
    #[Deprecated('Use getUserById instead')]
    public function getUser(int $id): User
    {
        // implementacja
    }
}
```

### Definiowanie własnych atrybutów
```php
#[Attribute]
class ApiEndpoint
{
    public function __construct(
        public string $path,
        public array $methods = ['GET']
    ) {}
}

#[ApiEndpoint('/api/users', ['GET', 'POST'])]
class UserApiController {}
```

### Odczytywanie atrybutów
```php
$reflectionClass = new ReflectionClass(UserController::class);
$attributes = $reflectionClass->getAttributes(Route::class);

foreach ($attributes as $attribute) {
    $instance = $attribute->newInstance();
    echo $instance->path; // '/users'
}
```

---

## Union Types

Pozwalają na deklarowanie, że parametr lub wartość zwracana może być jednym z kilku określonych typów.

### Przykłady użycia
```php
function processId(int|string $id): int|false
{
    if (is_string($id)) {
        return (int) $id;
    }
    return $id > 0 ? $id : false;
}

class User
{
    public function __construct(
        private string|null $email = null,
        private int|float $score = 0
    ) {}
}
```

### Złożone union types
```php
function handleResponse(array|object|null $data): string|array
{
    if ($data === null) {
        return 'No data';
    }
    
    if (is_array($data)) {
        return $data;
    }
    
    return json_encode($data);
}
```

---

## Match Expressions

Bardziej ekspresywna i bezpieczniejsza alternatywa dla instrukcji switch.

### Podstawowe użycie
```php
$result = match($status) {
    'pending' => 'Oczekuje',
    'approved', 'confirmed' => 'Zatwierdzono',
    'rejected' => 'Odrzucono',
    default => throw new InvalidArgumentException("Nieznany status: $status")
};
```

### Zaawansowane przykłady
```php
$httpMessage = match($response->getStatusCode()) {
    200, 201 => 'Success',
    400 => 'Bad Request',
    404 => 'Not Found',
    500 => 'Server Error',
    default => 'Unknown Status'
};

// Z wyrażeniami
$fee = match(true) {
    $amount < 100 => 0,
    $amount < 1000 => $amount * 0.02,
    $amount < 10000 => $amount * 0.015,
    default => $amount * 0.01
};
```

### Różnice względem switch
- Używa strict comparison (`===`)
- Nie wymaga `break`
- Zwraca wartość
- Wymaga obsłużenia wszystkich przypadków lub `default`

---

## Named Arguments

Umożliwiają przekazywanie argumentów według nazwy parametru, zwiększając czytelność kodu.

### Przykłady użycia
```php
function createUser(
    string $name, 
    string $email, 
    bool $isActive = true, 
    array $roles = []
): User {
    // implementacja
}

// Wywołanie z named arguments
$user = createUser(
    email: 'jan@example.com',
    name: 'Jan Kowalski',
    roles: ['admin', 'user']
);
```

### Kombinacja z pozycyjnymi argumentami
```php
// Pozycyjne argumenty muszą być przed named
$user = createUser(
    'Jan Kowalski',
    email: 'jan@example.com',
    roles: ['admin']
);
```

### Przydatne z funkcjami wbudowanymi
```php
htmlspecialchars(
    string: $userInput,
    flags: ENT_QUOTES | ENT_HTML5,
    encoding: 'UTF-8'
);

array_slice(
    array: $data,
    offset: 10,
    length: 5,
    preserve_keys: true
);
```

---

## Enumeracje (Enums)

*Wprowadzone w PHP 8.1*

Zapewniają typesafe sposób definiowania zestawów stałych wartości.

### Pure Enums
```php
enum Status
{
    case PENDING;
    case APPROVED;
    case REJECTED;
    case CANCELLED;
}

function updateStatus(Status $status): void
{
    match($status) {
        Status::PENDING => $this->sendNotification(),
        Status::APPROVED => $this->activateUser(),
        Status::REJECTED => $this->archiveRequest(),
        Status::CANCELLED => $this->cleanup()
    };
}
```

### Backed Enums
```php
enum HttpStatus: int
{
    case OK = 200;
    case NOT_FOUND = 404;
    case SERVER_ERROR = 500;
    case BAD_REQUEST = 400;
    
    public function message(): string
    {
        return match($this) {
            self::OK => 'Success',
            self::NOT_FOUND => 'Page not found',
            self::SERVER_ERROR => 'Internal server error',
            self::BAD_REQUEST => 'Bad request'
        };
    }
    
    public function isError(): bool
    {
        return $this->value >= 400;
    }
}
```

### String Backed Enums
```php
enum UserRole: string
{
    case ADMIN = 'administrator';
    case MODERATOR = 'moderator';
    case USER = 'regular_user';
    case GUEST = 'guest';
    
    public function getPermissions(): array
    {
        return match($this) {
            self::ADMIN => ['read', 'write', 'delete', 'manage'],
            self::MODERATOR => ['read', 'write', 'moderate'],
            self::USER => ['read', 'write'],
            self::GUEST => ['read']
        };
    }
}
```

### Użycie enums
```php
$status = HttpStatus::OK;
echo $status->value; // 200
echo $status->name; // 'OK'
echo $status->message(); // 'Success'

// Konwersja z wartości
$status = HttpStatus::from(404); // HttpStatus::NOT_FOUND
$status = HttpStatus::tryFrom(999); // null (bezpieczne)
```

---

## Constructor Property Promotion

Skrócona składnia dla konstruktorów eliminująca redundancję kodu.

### PHP 8 (nowa składnia)
```php
class User
{
    public function __construct(
        private string $name,
        private string $email,
        private bool $isActive = true,
        protected array $roles = []
    ) {}
    
    public function getName(): string
    {
        return $this->name;
    }
}
```

### PHP 7 (stara składnia)
```php
class User
{
    private string $name;
    private string $email;
    private bool $isActive;
    protected array $roles;
    
    public function __construct(
        string $name, 
        string $email, 
        bool $isActive = true,
        array $roles = []
    ) {
        $this->name = $name;
        $this->email = $email;
        $this->isActive = $isActive;
        $this->roles = $roles;
    }
    
    public function getName(): string
    {
        return $this->name;
    }
}
```

### Kombinacja z tradycyjnymi parametrami
```php
class Product
{
    private float $finalPrice;
    
    public function __construct(
        private string $name,
        private float $basePrice,
        float $taxRate = 0.23
    ) {
        $this->finalPrice = $this->basePrice * (1 + $taxRate);
    }
}
```

---

## Nullsafe Operator

Bezpieczne wywołania metod i dostęp do właściwości na potencjalnie null obiektach.

### Przykład użycia
```php
// Zamiast wielu sprawdzeń null
$country = null;
if ($session !== null) {
    $user = $session->user;
    if ($user !== null) {
        $address = $user->getAddress();
        if ($address !== null) {
            $country = $address->country;
        }
    }
}

// PHP 8 - nullsafe operator
$country = $session?->user?->getAddress()?->country;
```

### Praktyczne zastosowania
```php
// API responses
$userName = $apiResponse?->data?->user?->name ?? 'Unknown';

// Konfiguracja
$dbHost = $config?->database?->host ?? 'localhost';

// Łańcuchy metod
$result = $object?->method1()?->method2()?->method3();
```

---

## Mixed Type

Nowy typ reprezentujący dowolną wartość - alternatywa dla braku type hint.

### Przykłady użycia
```php
function processData(mixed $data): mixed
{
    return match(true) {
        is_array($data) => array_map('strtoupper', $data),
        is_string($data) => strtoupper($data),
        is_int($data) => $data * 2,
        default => $data
    };
}

class Cache
{
    private array $storage = [];
    
    public function set(string $key, mixed $value): void
    {
        $this->storage[$key] = $value;
    }
    
    public function get(string $key): mixed
    {
        return $this->storage[$key] ?? null;
    }
}
```

---

## Inne użyteczne nowości

### Throw Expressions
```php
$value = $input ?? throw new InvalidArgumentException('Input required');

$user = $users[$id] ?? throw new UserNotFoundException("User $id not found");
```

### str_contains(), str_starts_with(), str_ends_with()
```php
if (str_contains($email, '@')) {
    // zawiera @
}

if (str_starts_with($url, 'https://')) {
    // zaczyna się od https://
}

if (str_ends_with($filename, '.php')) {
    // kończy się na .php
}
```

### WeakMap
```php
$map = new WeakMap();
$object = new stdClass();
$map[$object] = 'some data';

// Gdy $object zostanie usunięty, wpis w WeakMap też zniknie
```

---

## Podsumowanie

PHP 8 wprowadził rewolucyjne zmiany, które znacząco poprawiają:

- **Type Safety** - Union types, mixed type, better type system
- **Czytelność kodu** - Named arguments, constructor promotion, match expressions
- **Bezpieczeństwo** - Nullsafe operator, strict comparisons w match
- **Produktywność** - Mniej boilerplate kodu, bardziej ekspresywna składnia
- **Nowoczesność** - Atrybuty, enumeracje, nowe funkcje string

Te funkcje czynią PHP bardziej konkurencyjnym względem nowoczesnych języków programowania, zachowując przy tym prostotę i elastyczność, za które PHP jest ceniony.

---

## Przydatne linki

- [PHP 8.0 Release Notes](https://www.php.net/releases/8.0/en.php)
- [PHP 8.1 Release Notes](https://www.php.net/releases/8.1/en.php)
- [PHP Manual - Enumerations](https://www.php.net/manual/en/language.enumerations.php)
- [PHP Manual - Attributes](https://www.php.net/manual/en/language.attributes.php)
