---
layout: :theme/post
title: "Compensating Transactions: When ACID is too much (Part 4: Long Lived Transactions)"
description: This is my first article ever made with Quarkus Roq
tags: transactions
author: paul
---

In [part one in this series]({site.url('posts/compensating-transactions-when-acid-is-too-much-part-1-introduction')}) I explained how ACID transactions can have a negative impact on applications whose transactions can take a relatively long time to run.
In addition, another potential issue with ACID transactions is that the failure of one unit can cause the entire transaction to be rolled back.
This is less of an issue for short running transactions, as the previously successful work can be retried quickly.
However, for long-running transactions, this previously completed work may be significant, leading to a lot of waste should it need to be rolled back.
In this post, I'll show how a compensation-based transaction could be a better fit for long-lived transactions.

A compensation-based transaction can be composed of multiple short-lived ACID transactions.
When each transaction completes, it releases the locks on the resources it held, allowing other transactions requiring those resources to proceed.
A compensation action is registered for each ACID transaction and can be used to undo any work completed, should the entire compensation-based transaction need to be aborted.
Furthermore, should one of these short-lived ACID transactions fail, it could be possible to find an alternative, preventing the entire transaction from failing.
This allows forward progress to be achieved.
By composing the compensation-based transaction as several units of work, you also gain the opportunity to selectively abort (compensate) particular units as the compensation-based transaction progresses.
A simple example should help to clarify these ideas...

For example, take a travel booking scenario.
We begin by booking a flight.
We then try to book a taxi, but that fails.
At this point we don’t want to compensate the flight as it may be fully-booked next time we try.
Therefore, we try to find an alternative Taxi which in this example succeeds.
Later, in the compensation-based transaction, we may find a cheaper flight.
In which case we want to cancel the original flight whilst keeping the taxi and the cheaper flight.
In this case we notify our intentions to the transaction manager who ensures that the more expensive flight is compensated when the compensation-based transaction completes.

# Code Example
In this example, I expand on the Travel Agent example from part 3 in this series.
Here I will show how a failure to complete one unit of work does not have to result in the whole compensation-based transaction being aborted.

```
public class Agent {

    @Inject
    HotelService hotelService;
    
    @Inject
    Taxi1Service taxi1Service;
    
    @Inject
    Taxi2Service taxi2Service;
    
    @Compensatable
    public void makeBooking(String emailAddress, String roomType, String destination) throws BookingException {
    
        hotelService.makeBooking(roomType, emailAddress);
        
        try {
            taxi1Service.makeBooking(destination, emailAddress);
        } catch (BookingException e) {
            /**
            * Taxi1 rolled back, but we still have the hotel booked.
            We don't want to lose it, so we now try Taxi2
            */
            taxi2Service.makeBooking(destination, emailAddress);
        }
    }
}
```

For this example, you can imagine that the Hotel and Taxi services are implemented similarly to the HotelService in part 3.
The `makeBooking` method is annotated with `@Compensatable`, which ensures that the method is invoked within a compensation-based transaction.
The method begins by making a Hotel reservation.
If this fails, we don't handle the BookingException, which causes the compensation-based transaction to be canceled.
If the hotel booking succeeds, we then move onto booking a taxi.
If this particular booking fails, we catch the `BookingException` and try an alternative Taxi company.
Because the Taxi service failed immediately, we know that it should (it's a requirement of using this API) have undone any of it's work.
We can therefore choose to fail the compensation-based transaction or, in this case, try an alternative Taxi company.
The important thing to note here is that we still have the hotel booked, and we don't want to lose this booking as the hotel may be fully booked next time we try.
The code then goes on to attempt an alternative taxi company.
If this booking fails, we have no option but to cancel the whole transaction as we have no other alternatives.

# Conclusion
In this blog post, I showed how a compensation-based transaction could be a good fit for long-running transactions.