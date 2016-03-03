---
layout: page
title: In search for a perfect return
permalink: /search-for-return/
---

I like to separate all the methods that I write into 3 logical parts:

`collect input` -> `process data` -> `provide output`

I have found that collecting input and providing output may become very repetitive and often even duplicated.

The task that really bothers me is providing a result of the method that can be either used to make a subsequent decision or to be subsequently consumed by another method.  

Let me illustrate:

```ruby
def make_payment
  ..process and execute payment
end
```

<!--break-->

Very often we will need to assert the result to make a decision to continue further or not:

`if make_payment then ..` or `return unless make_payment`

So, in this case, returning a boolean will do the job.

If payment has succeeded, we might need to feed the result of the payment into another method.

```ruby
if (payment = make_payment) then register_payment(payment)
```

Ok, by relying on ruby's truthiness, we can return nil or payment result, e.g. `PaymentObject`.
This will do  the job, but raises a few issues.

From the point of view of statically typed languages, you should always return the same type. In this case, it can be a `NilObject` or `PaymentObject`.  Of course, ruby is more flexible, but it makes sense, because if you do not know the return type, then you should always be prepared for either of them, which means asserting the type of the result, in one way or another (and, in essence any conditional is basically doing `(..).is_a?(TrueClass)` ).  

Now, functional languages have a wonderful concept of `SOME` (aka `MAYBE/EITHER`), which is a container that can hold either a value of some type or a nil (null, none, etc.).   So, whenever you see this type, you know that the result could have been nil and you process it accordingly. (In fact, some languages like ML enforce that you pattern match both cases)  

In ruby, we use a `NullObject` pattern for similar reasons - signal that the outcome could have been nil and provide means for procesing ressults in a uniform passion, regardless of the actual outcome and avoiding conditionals.

```ruby
def make_payment(params)
  if(success)
    NullPaymentObject.new
  else
    PaymentObject.new
  end
end

register_payment(make_payment(params))
```

Since, both `NullPaymentObject` and `PaymentObject` have the same interface, we can just feed them on the next method, without thinking too much about the contents and, thus, eliminating the conditional.  (In fact, we can even wrap the result in a `GuaranteedPaymentObject` to return the same type)

However, the NullObject pattern is more appropriate in situations, when we do not need to stop execution, e.g.  `display_user(get_current_user)`

Here, we would still need to display somehting, even if there is no current user.

In the original example, we actually need to stop execution if a payment failed.  In this case, we would still need an assertion of some kind, so returning nil is probably the most sensible solution, since a NullObject will be truthy in a conditional and things may become tricky (although, this hiccup can be dealt with).

There is another inconvenience: most often we would like to know, why the method failed, at least for logging purposes.  It would be nice, if our result carried some information with it.  

This brought me to the idea of returning a `StatusObject`, which would carry the result of the method execution in case of success and reasons for failure, in case of failure.

```ruby
class StatusObject
  def initialize()
    @status = :success
    @result = nil
    @error = {}
  end

  def success!(result)
    @status = :success
    @result = result
  end

  def fail!(error_details = {})
    @status = :failure
    @error = error_details
  end
end
```

Now, we can use it like this:

```ruby
def make_payment(params)
  status = StatusObject.new

  if(..payment succeeded..)
    status.success!(result_of_method)
  else
    status.fail!(reasons_for_failure)
  end
end

status = make_payment(payment_params)

if status.success?
  make_payment(status.result)
else
  log status.errors
end
```

or even like this:

```ruby
def make_payment(params)
  status = StatusObject.new
  (..payment succeeded..) ?  status.success!(result_of_method) : status.fail!(reasons_for_failure)
end

status = make_payment(payment_params)

status.success? ?  register_payment(status.result) : log(status.errors)
```

or how about this:

```ruby
 make(payment)
  .on_success { |status, result| register_payment(result) }
  .on_failure { |status, error| log(error) }
```

To be continued...
