---
title: 'SRP in a Laravel Project'
description: 'In this article I will talk about the first principle in SOLID.'
pubDate: 'Aug 21 2023'
heroImage: '/solid.png'
---

This principle declare that a class must be specialized in a single subject and have a single responsibility within the software, in other words, a class must have a single or action to perform.

Violate that principle can bring some problems such as:

- Lack of cohesion
- High coupling
- Difficulties reusing the code
- Difficulty to implement automated tests

#### Example

```php
class UserController extends Controller
{
    public function register(Request $request): JsonResponse
    {
        $request->validate([ // Request validation
            'name'  => ['required', 'string'],
            'email' => ['required', 'email'],
            'role'  => ['required', new Enum(UserRoles::class)],
        ]);
    
        $user = new User();
        $user->name = $request->input('name');
        $user->email = $request->input('email');
        $user->role = $request->input('role');
        $user->password = Hash::make($request->input('password'));
        $user->save(); // Interact with daatabase

        if ($user->role === 'customer') { // Business Rule
            Mail::to($user->email)
            ->send(new WelcomeCustomerEmail($user)); // Email Dispatch
        }

        return response()->json(['id' => $user->getKey()], 201);
    }
}
```

As we can see above, the class `UserController` has several responsibilities:

- Validate request body
- Save user in the database
- Business Rule
- Send welcome email if user is customer

When actually, the function of a controller is just receive requests and return responses. We must separate these responsibilities into other layer, such as:

##### Layers

- Requests validators (In laravel you can use **Form Requests**, but in other frameworks you can create some middlewares)
- Controller - ( Receive requests and return response )
- Repository - ( Interact with database )
- Service - ( Business Rule )


#### Request validator (Form Request)

```php
class UserRegisterRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name'  => ['required', 'string'],
            'email' => ['required', 'email'],
            'role'  => ['required', new Enum(UserRoles::class)],
        ];   
    }
}

```

#### Controller

```php
class UserController extends Controller
{
    public function __construct(
        protected UserService $userService
    ) {
    }
	
    public function register(RegisterUserRequest $request): JsonResponse
    {
        $dto = new RegisterUserDto($request->all()); 
        
        $user = $this->userService->register($dto);
        
        return response()->json(['id' => $user->getKey()], Response::HTTP_CREATED);
    }
}
```
I use a **RegisterUserDto**, where 'Dto' stands for 'Data Transfer Object.' This is employed when transferring data across different layers.

#### Service
```php
class UserService
{
    public function __construct(
       protected UserRepository $userRepository,
    ) {
    }
    
    public function register(RegisterUserDto $dto): User
    {
        $user = $this->userRepository->save($dto);
        
        if ($user->isCustomer()) {
            $this->sendWelcomeEmail($user);        
        }
        
        return $user;
    }
    
    protected function sendWelcomeEmail(User $user): void
    {
        Mail::to($user->email)->send(new WelcomeUserEmail($user));
    }
}
```
#### Repository

```php
class UserRepository
{
    public function save(RegisterUserDto $dto): User
    {
        return User::create($dto->toArray());
    }
}
```

Now that project is more flexible to implement new features, your code became more readable, in tests, now you can easily utilize mocks for your repository, for example, so you don't need to interact with the database, which would slow down your tests.

#### Tip

If you application use the pattern to return created responses with entity id, you can add a helper method in your Base Controller.



##### Base Controller
```php
class Controller extends BaseController
{
    protected function created(int $id): JsonResponse
    {
        return response()->json(['id' => $id], Response::HTTP_CREATED);
    }    
}
```

##### UserController
```php
class UserController extends Controller
{
    public function __construct(
        protected UserService $userService
    ) {
    }
	
    public function register(RegisterUserRequest $request): JsonResponse
    {
        $dto = new RegisterUserDto($request->validated()); 
        
        $user = $this->userService->register($dto);
        
        return $this->created($user->getKey());
    }
}
```