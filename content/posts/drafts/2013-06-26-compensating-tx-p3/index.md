---
layout: :theme/post
title: "Compensating Transactions: When ACID is too much (Part 3: Cross-Domain Distributed Transactions)"
description: This is my first article ever made with Quarkus Roq
tags: transactions
author: paul
---

In [part one in this series]({site.url('posts/compensating-transactions-when-acid-is-too-much-part-1-introduction')}) I explained why ACID transactions are not always appropriate.
I also introduced compensation-based transactions as a possible alternative to ACID transactions.
In this post I'll show how compensation-based transactions could be a better fit, than ACID transactions, for distributed applications that cross high latency networks or span multiple business domains.

When your application becomes distributed, and more systems become involved, you inevitably increase the chances of failure.
Many of these failures can be tolerated using a distributed ACID transaction.
However, an ACID transaction can be seen as impractical for certain distributed applications.
The first reason for this, is that distributed transactions that cross high latency networks (such as the Internet), can take a relatively long time to run.
As I showed in [part 1]({site.url('posts/compensating-transactions-when-acid-is-too-much-part-1-introduction')}) , increasing the time to run an ACID transaction can have negative impacts on your application.
For example, the holding of database resources for prolonged periods can significantly reduce the throughput of your application.
The second reason is due to the tight coupling between the participants of the transaction.
This tight coupling occurs because the root coordinator of the transaction ultimately drives all the transactional resources through the 2PC protocol.
Therefore, once prepared, a transactional resource has to either make a heuristic decision (bad) or wait for the root coordinator to inform it of the outcome.
This tight-coupling may be acceptable if your distributed application resides in a single business domain where you have control of all the parties.
However, it is less likely to be acceptable for distributed applications that span multiple business domains.

Compensation-based transactions could prove to be a better solution for these scenarios.
As I showed in [part 1]({site.url('posts/compensating-transactions-when-acid-is-too-much-part-1-introduction')}), compensation-based transactions can be more suitable for longer lived transactions, as they don't need to hold onto database resources until the transaction completes.
Compensation-based transactions can also be used to decouple the back-end resources from the transaction coordinator.
This can be done by splitting the two phases of the protocol into abstract business operations, such as book/cancel.
The 'book' operation makes an update to the database to create the booking.
As this update is committed immediately, there is no tight-coupling between the database resources and the transaction coordinator.
The cancel operation is invoked by the compensation handler, should the compensation-based transaction need to abort.

I'll also show through this example, how isolation can be preserved whilst moving the resource locking from the database-level up to the application-level where it is easier for the application to reason about.

# Code Example
In this example we'll look at a simple travel booking example. Here a client makes a hotel and taxi booking (with remote services) inside a compensation-based transaction.
These remote services are hosted in different business domains and are invoked over the Internet.

```
public class Client {

    @Compensatable
    public void makeBooking() throws BookingException {

        // Lookup Hotel and Taxi Web Service ports here...
 
        hotelService.makeBooking("dbl", "paul.robinson@redhat.com");
        taxiService.makeBooking("ncl", "paul.robinson@redhat.com");
    }
}
```

This code forms part of the client application.
The 'makeBooking' method is annotated with `@Compensatable` which ensures that the method is invoked within a compensation-based transaction.
The method invokes two Web services.
These Web services support WS-BA, so the transaction is transparently distributed over these calls.

```
@WebService
public class HotelService {

    @Inject
    BookingData bookingData;

    @Compensatable(MANDATORY)
    @TxCompensate(CancelBooking.class)
    @TxConfirm(ConfirmBooking.class)
    @Transactional(value=REQUIRES_NEW, rollbackOn=BookingException.class)
    @WebMethod
    public void makeBooking(String item, String user) throws BookingException {
    
        //Update the database to mark the booking as pending...
    
        bookingData.setBookingID("the id of the booking goes here");
    }
}
```

Above is the code for the Hotel's Web Service.
As well as the usual JAX-WS annotations that you would expect (some omitted for brevity), there are some extra annotations to manage the transactions.
The first is `@Compensatable(MANDATORY)`; this ensures that this method is invoked within the scope of a compensation-based transaction.
The second annotation (`@TxCompensate`) provides the compensation handler, which you should be familiar with from Part 2 in this series.
The third annotation (`@TxConfirm`), may be new to you.
This annotation provides a handler that is invoked at the end of the transaction if it was successful.
This allows the application to make final changes once it knows the transaction will not be compensated.
Hopefully, the need for this feature will become more clear as we discuss this example further.
Finally, this example uses the JTA 1.2 `@Transactional` annotation to begin a new JTA transaction.
This transaction is used to make the update to the database and will commit if the `makeBooking` method completes successfully.
The JTA transaction will rollback if a BookingException (see the rollbackOn attribute) or a RuntimeException (or a subclass of) are thrown.

So, why does the JTA transaction commit at the end of this method call even though the compensation-based transaction is still running?

This is done to reduce the amount of time the service holds onto database resources.
The application could simply add the booking to the database, but it's possible that at some time in the future it might need to be canceled.
Therefore in this example, the application just marks the booking as pending.
Therefore, any other transaction that reads the state of the bookings table will see that this particular booking is tentative.
Remember, by using a compensation-based transaction, we have relaxed isolation, and this is one way in which the application can be modified to tolerate this.

```
public class ConfirmBooking implements ConfirmationHandler {

    @Inject
    BookingData bookingData;
    
    @Override
    @Transactional(REQUIRES_NEW)
    public void confirm() {
        //Confirm order for bookingData.getBookingID() in Database (in a JTA transaction)
    }
}
```

As mentioned above, the ConfirmationHandler provides a callback that occurs when the compensation-based transaction completes successfully.
In this example, the confirmation handler begins a new JTA transaction and then updates the database to mark the booking as finalized.

```
public class CancelBooking implements CompensationHandler {

    @Inject
    BookingData bookingData;
    
    @Override
    @Transactional(REQUIRES_NEW)
    public void compensate() {
        //Cancel order for bookingData.getBookingID() in Database (in a new JTA transaction)
    }
}
```

Similarly, above is the compensation handler that starts a new JTA transaction which cancels the booking.

In this example we are essentially making a trade-off between 2 shorter lived JTA transactions, with application-level locking (in this example) in place of 1 longer lived JTA transaction with database-level locking (had we used a distributed JTA transaction).
The isolation level is roughly the same for both cases.
We also needed to make a change to the application.
Whether this is a sensible trade-off will depend largely on your application and throughput requirements.

It is also worth noting that the middleware can (we don't yet implement this, see [here](https://community.jboss.org/thread/220541) and [here](https://issues.jboss.org/browse/JBTM-1099)) reliably tie the compensation-based and the JTA transaction together.
It does this by preparing the JTA transaction when the method completes, but holds off committing it until the compensation handler has been logged to the transaction log.
This ensures that the work is only committed if it can later be compensated in the case of failure.
The invocation of the CompensationHandler and the ConfirmationHandler is also reliable as they are invoked repeatedly until they complete successfully.
Therefore, it is important for their implementations to be idempotent.