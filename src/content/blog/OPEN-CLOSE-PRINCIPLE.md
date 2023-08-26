---
title: 'Open Closed Principle'
description: 'In this article I will talk about the open close principle from SOLID.'
pubDate: 'Aug 26 2023'
heroImage: '/pingu3.jpg'
---

This principle states that your class is closed for modifications but open for extensions. In other words, when implementing new features and behaviors in software, the goal is to extend the codebase without altering the original code.


Let's support that we have payments in our application, we have multiple payment methods.

For example:

```php
class Payment
{
    public function payWithCreditCard(Invoice $invoice): Payment
    {
        // Logic for paying with a credit card
    }
    
    public function payWithPayPal(Invoice $invoice): Payment
    {
        // Logic for paying with PayPal
    }
    
    public function payWithSlip(Invoice $invoice): Payment
    {
        // Logic for paying with a slip
    }
}
```

#### Usage

```php
class PaymentService
{
    public function pay(Invoice $invoice): void
    {
        $payment = new Payment();
    
        if ($invoice->payment_method === 'credit_card') {
            $payment->payWithCreditCard($invoice);
            return;
        }
        
        if ($invoice->payment_method === 'paypal') {
            $payment->payWithPayPal($invoice);
            return;
        }
        
        if ($invoice->payment_method === 'slip') {
            $payment->payWithSlip($invoice);
            return;
        }
    }    
}
```

In this case if we need to implement a new payment method we have to change our codebase, which is a bad practice.

#### Using Open Closed Principle

First of all, we abstract the payment method, in this case we gonna use an interface.

```php
interface PayableInterface
{
    public function pay(Invoice $invoice): Payment;
}
```
Now we guarantee that all payable classes will have the method `pay` that receive an `Invoice` and will return a `Payment`, we don't have to know which class will implement that interface, because we know its methods and funcionalities.



#### Implementations

```php
class CreditCardPayment implements PayableInterface
{
    public function pay(Invoice $invoice): Payment
    {
        // Logic for payment with credit card
    }
}

class PayPalPayment implements PayableInterface
{
    public function pay(Invoice $invoice): Payment
    {
        // Logic for payment with paypal
    }
}

class SlipPayment implements PayableInterface
{
    public function pay(Invoice $invoice): Payment
    {
        // Logic for payment with slip
    }
}
```
Now we have better separation, we have more control of each payment method, we can easily create a new payment method or change something in an existing one.

#### Enums
Enums are a valuable recommendation, particularly when dealing with diverse values for a specific property of a model.

```php
enum PaymentMethod: string
{
    case PAYPAL = 'paypal';    
    case SLIP = 'slip';    
    case CREDIT_CARD = 'credit-card';
    // ...    
}
```

Don't forget to use the `$casts` in your model.

```php
class Invoice extends Model
{
    protected $casts = [
        'payment_method' => PaymentMethod::class,
    ];
}
```
Now let's take a look how the usage becomes...

#### Usage

```php
class PaymentService
{
    public function pay(Invoice $invoice): Payment
    {
        $payable = match($invoice->payment_method) {
            PaymentMethod::SLIP => new SlipPayment(),
            PaymentMethod::PAYPAL => new PayPalPayment(),
            PaymentMethod::CREDIT_CARD => new CreditCardPayment(),
            default => throw new InvalidArgumentException('Unsupported payment method')
        }
    
        return $payable->pay($invoice);
    }
}
```

Nice, now this code becomes better, but we can introduce a new concept called **Factories**, when we have this logic for which concrete class will be instantiated, we have factories for this, lets take a look at how that works using factory pattern.


#### Factory
```php
class PayableFactory
{
    public static function new(PaymentMethod $method): PayableInterface
    {
        return match($method) {
            PaymentMethod::SLIP => new SlipPayment(),
            PaymentMethod::PAYPAL => new PayPalPayment(),
            PaymentMethod::CREDIT_CARD => new CreditCardPayment(),
            default => throw new InvalidPaymentMethodException(),
        }
    }
}
```


 

#### Final Usage

```php
class PaymentService
{
    public function pay(Invoice $invoice): Payment
    {
        $payable = PayableFactory::new($invoice->payment_method);
    
        return $payable->pay($invoice);
    }
}
```

Now it becomes much easier to implement a new payment method, we just have to create a new concrete class and put the logic inside that class, we don't change any other payment method.
 