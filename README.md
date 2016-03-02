# yarospace.github.io

My programming notes
==

I am one of those people who believe that programming is their passion.  Moreover, among many other things I find programming to be an art, which allows individuals to express their vision of the world in code and experience aesthetic pleasure, both from the process of writing code and the end result.

I have tried a few programming languages and paradigms and I am constantly in the search of yet a better way to express myself.  So far, my language of choice is Ruby and I am deeply influenced by SmallTalk and functional approach to programming.   I like to improve in everything I do and this blog is my reflection and thoughts about these improvements.

Sometimes, I have a feeling that I am a good programmer, but I a quick glance at the work of my greater colleagues is  enough to give me a cold shower and to remind me that I am the very beginning of the road.  Thus, your constructive criticism and advice is very welcome.

February 2016 - In search for perfect return
===

I like to separate all the methods that I write into 3 logical parts: collect input, process data, provide output.

I have found that these tasks may become very repetitive and similar, sometimes even duplicated.  One task that really bothers me is providing a result of the method that can be either used to make a subsequent decision or to be subsequently consumed by another method.  

Let me illustrate:

```
def make_payment
  ..process and execute payment
end
```

Very often we will need to assert the result to make a decision to continue further or not:

`if make_payment then ..`

or

`return unless make_payment`

So, in this case, returning a boolean will do the job.

If payment has succeeded, we might need to feed the result of the payment into another method.

`if (payment = make_payment) then register_payment(payment)`

Ok, by relying on ruby's truthiness, we can return nil or payment result, e.g. `PaymentObject`.
This will do  the job, but raises a few issues.

From the point of view of statically typed languages, you should always return the same type. In this case, it can be a `NilObject` or `PaymentObject`.  Of course, ruby is more flexible, but it makes sense, because if you do not know the return type, then you should always be prepared for either of them, which means asserting the type of the result, in one way or another (and, in essence any conditional is basically doing `(..).is_a?(TrueClass)` ).  

Now, functional languages have a wonderful concept of `SOME` (aka `MAYBE/EITHER`), which is a type that can hold either a value of some type or a nil (null, none, etc.).   So, whenever you see this type, you know that the result could have been nil and you process it accordingly. (In fact, some languages like ML enforce that you pattern match both cases)  

In ruby, we use a `NullObject` pattern for similar reasons - signal that the outcome could have been nil and provide means for procesing ressults in a uniform passion, regardless of the actual outcome and avoiding conditionals.

```
def make_payment(params)
  if(success)
    return NullPaymentObject
  else
    return PaymentObject
  end
end

register_payment(make_payment(params))
```

Since, both `NullPaymentObject` and `PaymentObject` have the same interface, we can just feed them on the next method, without thinking too much about the contents and, thus, eliminating the conditional.  (In fact, we can even wrap the result in a `GuaranteedPaymentObject` to return the same type)

However, the NullObject pattern is more appropriate in situations, when we do not need to stop execution, 

e.g. `display_user(get_current_user)`.  

Here, we would still need to display somehting, even if there is no current user.  

In the original example, we actually need to stop execution if a payment failed.  In this case, we would still need an assertion of some kind, so returning nil is probably the most sensible solution, since a NullObject will be truthy in a conditional and things may become tricky (although, this hiccup can be dealt with).

There is another inconvenience: most often we would like to know, why the method failed, at least for logging purposes.  It would be nice, if our result carried some information with it.  
This brought me to the idea of returning a `StatusObject`, which would carry the result of the method execution in case of success and reasons for failure, in case of failure.

```
class StatusObject
  def initialize()
    @status = :success
    @result = nil
    @error = {}
  end

  def success!(result)
    @result = result
  end

  def fail!(error_details = {})
    @error = error_details
  end
end

def make_payment(params)
  status = StatusObject.new

  if(..payment succeeded..)
    return status.success!(result_of_method)
  else
    return status.fail!(reasons_for_failure)
  end
end

status = make_payment(payment_params)

if status.success? 
  make_payment(status.result)
else
  log status.errors
end
```

To be continued...
