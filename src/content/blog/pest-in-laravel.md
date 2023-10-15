---
title: 'Pest in Laravel project'
description: 'How to use Pest php in your Laravel project'
pubDate: 'Oct 15 2023'
heroImage: '/pingu-pest.jpg'
---
The **Pest** is a testing Framework that gained his fame for his simplicity and elegance.

That tool is being widely adopted in Laravel projects, so I'm writing this content to bring some ways to write tests using Pest applied to a project that uses Laravel.

Of course, firstly I will introduce the anatomy of Pest and how to write your first test.

### Introduction of Pest

<br>

##### Your first test using Pest

After you install Pest and configure your project, you're ready to write your first test.

To write tests using pest you can use the method **"it"**

**Example:**

```php
// FirstTest.php


it('your test description here', function () {
    // Your expectations here
});
```

```php
// SumTest.php

function sum(int $num1, int $num2): int {
    return $num1 + $num2;
}

// Your test
it('should sum two numbers', function () {
    $result = sum(2, 6);
    
    expect($result)->toBe(8);
});
```

The **expect()** method provides several assertions that we can perform.

You can check a list of them <a href="https://pestphp.com/docs/expectations" target="_blank">here</a>.

##### The Test Anatomy (AAA)

Generally we can separate our test in **3** parts, being the famous **Triple A (AAA)**, which means: (**Arrange**, **Act**, **Assert**):

**Arrange:**
Here we will prepare our test, initialize all variables, classes and data which will be used for the test.

**Act:**
Where we will excecute the action that our test are asserting/testing.

**Assert:**
From the result of the last step, we will check if what we got is what was expected.

**Example:**

```php
// SumTest.php

function sum(int $num1, int $num2): int {
    return $num1 + $num2;
}

it('should sum two numbers', function () {
    // Arrange
    $num1 = 6;
    $num2 = 2;
    $expectedResult = 8;
    
    // Act
    $result = sum($num1, $num2);
    
    // Assert
    expect($result)->toBe($expectedResult);
});
```

In most of the cases we can identify this pattern.

##### Hooks

The Pest like others testing frameworks has the concept of **Hooks**, which
are methods that you can perform some action at certain times of your test.

**Pest hooks:**
- beforeEach
- afterEach
- beforeAll
- afterAll

<br>

**beforeEach:**
Is executed **before** the execution of **each** test in your file.

Used to initialize data that will be used by all your tests in that file.

**Example:**

```php
// SumTest.php

beforeEach(function () {
    $this->calculator = new Calculator();
});
 
it('should sum two numbers', function () {
    $result = $this->calculator->sum(6, 2);
    
    expect($result)->toBe(8);
});
```

<br>

**afterEach:**
Is executed **after** **each** test in that file be completed.

Used to reset data that was created by the previous test, to avoid interference.

**Example:**

```php
// SumTest.php

beforeEach(function () {
    $this->calculator = new Calculator();
});

afterEach(function () {
    $this->calculator->reset();
});
 
it('should sum two numbers', function () {
    $result = $this->calculator->sum(6, 2);
    
    expect($result)->toBe(8);
});
```
**beforeAll:**
It's important to remember that this hook as well as **afterAll**, doesn't have acess to **$this**.
Unlike hooks: **beforeEach, afterEach**.

It's executed **before** the execution of **all** tests in that file.

**afterAll:** It's executed **after** the execution of **all** tests in that file.

##### Testing Markings

Let's say you have written a sequence of tests cases, but you are not going to finish them all yet.

In Pest you can mark them using: **skip** or **todo** method.

**Example:**

```php
// CalculatorTest.php

it('should sum two decimal numbers')->todo();

it('should multiple two numbers')->skip();

it('should divide two numbers')->todo();
```

It's very usefull mark your tests, that way you or your team will be aware that this case must be finished. 

##### Pest config file

The Pest configuration file (**tests/Pest.php**). It is possible to carry out some actions that help our organization and writing of tests.

```php
// tests/Pest.php

uses(
    Tests\TestCase::class, 
    RefreshDatabase::class,
)->in('Feature');

expect()->extend('toBeOne', function () {
    return $this->toBe(1);
});

function something()
{
    //
}
```

<br>

##### Class, Trait Binding

In this config file you can assign classes and traits to differente folders in your project.

**Example:**

```php
uses(
    Tests\TestCase::class,
    RefreshDatabase::class,
)->in('Feature');
```
In this example we're declaring that all our tests in the **tests/Feature** folder will run on top of the class **Tests/TestCase.php**, and will use **RefreshDatabase** trait.

So is also possible use the **uses()** method inside your test file for a specific configuration.

##### Custom excpectations

We often need to make custom assertions, and in **Pest** we have a very simple way to create this.

**Example:**

```php
// Pest.php
expect()->extend('toBeOne', function () {
    return $this->toBe(1);
});

expect()->extend('isNow', function () {
    return $this->toEqual(now());
});

expect()->extend('toBeInRange', function (int $min, int $max) {
    return $this
        ->toBeGreatherThan($min)
        ->toBeLessThan($max);
});
```
As you can see, the **$this**, it will always be the value that is being "**expected**".

**Example:**

```php
it('should be one', function () {
    expect(1)->toBeOne();
})->freezeSeconds();

it('should be now', function () {
    expect(now())->isNow();
})->freezeSeconds();

it('should be in range', function() {
    expect(5)->toBeInRange(1, 10);
});
```
<br>

##### High order testing

In Pest, the way it was built, 
allows us to write tests just using its methods,
without the need to add a closure.

**Example:**

```php
it('should be now', function () {
    expect(now())->isNow();
})->freezeSeconds();
```

Using high order testing, our test can be rewritten like this.

```php
it('should be now')
    ->expect(now())
    ->isNow()
    ->freezeSeconds();
```

And in several other test scenarios we can use this pattern.

**Requests tests**

```php
// Classic way
it('should return status ok', function () {
    $response = $this->get('/');
    
    $response->assertstatus(200);
});

// High order testing
it('should return status ok')
    ->get('/')
    ->assertOk();
```
Another interesting tip, if you create functions in the same namespace, you can use it in this sequence of functions.

```php
function makeRequest(): TestResponse
{
   $user = User::factory()->create();
   
   return get("user/{$user->getKey()}")
}

it('should find user sucessfully')
    ->makeRequest()
    ->assertOk();
```

<br>

##### Helper functions

Another very common scenario in several projects, is the need to create global functions that will help our testing.

In **Pest** we can create this in **test/Pest.php** configuration file


**Example:**

```php
// PostTest.php -- Before

it('user can create post when logged in', function () {
    $user = User::factory()->create();
    
    $this->actingAs($user);
    
    $payload = [
        'title' => fake()->sentence,
        'content' => fake()->text,
    ];
    
    $response = $this->post('/posts', $payload);
    
    $response->assertStatus(201);
});
```

```php
// tests/Pest.php

function user(array $state = []): User {
    return User::factory()->state($state)->create();
}

function loggedAs(User $user = null) {
    $user ??= user();

    actingAs($user);
}
```

```php
// PostTest.php -- After 

function makeRequest(array $payload = []): TestResponse {
    return post('/posts', [
        'title' => fake()->sentence,
        'content' => fake()->text,
        ...$payload,
    ]);
}

it('user can create post when logged in')
    ->loggedAs()
    ->makeRequest()
    ->assertCreated();
```

<br>

##### Datasets

With **Datasets** you can create a sequence of datas that allows your tests to execute with differents types of values.

**Example:**

Let's assume that we are creating an rule that just allows the payment of a invoice when the invoice status is **Open**.

```php
it('should not pay an invoice when status is not opened', function () {
    $invoice = Invoice::factory()
        ->state(['status' => InvoiceStatus::CANCELED])
        ->create();
    
    $invoice->pay();
})->throws(InvalidInvoiceException::class);
```

In this scenario our test will pass, but a bug may still be occurring when a status is other than canceled, such as:

- Refunded
- Paid

Writing a test for each of these values would be very verbose, **Datasets** came to solve this.

```php
it('should not pay an invoice when status is not opened', function (InvoiceStatus $status) {
    $invoice = Invoice::factory()
        ->state(['status' => $status])
        ->create();
    
    $invoice->pay();
})->with([)
    ->throws(InvalidInvoiceException::class);
```

Now, our test will run for each value passed for **with** method.

We can make this better, let's assume that other tests needs this sequence of status, we can move that values to other file.

Inside the **tests/Datasets** folder we can create the file **InvoiceDatasets.php** with:

```php
datasets('not opened invoice status', [
    InvoiceStatus::CANCELED,
    InvoiceStatus::PAID,
    InvoiceStatus::REFUND,
]);
```
Now in our test we can use this alias

```php
it('should not pay an invoice when status is not opened', function (InvoiceStatus $status) {
    $invoice = Invoice::factory()
        ->state(['status' => $status])
        ->create();
    
    $invoice->pay();
})->with('not opened invoice status')
    ->throws(InvalidInvoiceException::class);
```
