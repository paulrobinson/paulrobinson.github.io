---
layout: :theme/post
title: "Compensating Transactions: When ACID is too much (Part 2: Non-Transactional Resources)"
description: This is my first article ever made with Quarkus Roq
tags: transactions
author: paul
---

In [part one in this series]({site.url('posts/compensating-transactions-when-acid-is-too-much-part-1-introduction')}) I explained why ACID transactions are not always appropriate.
I also introduced compensation-based transactions as a possible alternative to ACID transactions.
In this post I'll focus on situations where the application needs to coordinate multiple non-transactional resources and show how a compensation-based transaction could be used to solve this problem.

For the sake of this discussion, I'm defining a transactional resource as one that can participate in a two phase protocol and can thus be prepared and later committed or rolled back.
For example, XA-capable databases or message queues would be considered transactional resources.
In contrast, a non-transactional resource is one that does not offer this facility.
For example, the sending of an email or printing of a cheque can not easily participate in this two phase protocol. 
Third party services can also be hard to coordinate in an ACID transaction.

Even though these services might be implemented with ACID transactions (under the covers), they might not allow participation in an existing transaction.
A compensation-based transaction could be a good fit for these situations too.
The non-transactional work can be carried out in the scope of the compensation-based transaction.
Providing that a compensation handler is registered, the work can later be undone, should the compensation-based transaction need to be aborted.
For example, the compensation handler for sending an email could send a second email asking the recipient to disregard the first email.
The printing of a cheque could be compensated by canceling the cheque and notifying the recipient of the cancellation.

It's also possible to coordinate transactional and non-transactional resources in a compensation-based transaction.
Here the application just needs to create compensation handlers for the non-transactional resources.
You could still use an ACID transaction with the last resource commit optimization (LRCO) if you only have one non-transactional resource, but this approach is not recommended if you have multiple non-transactional resources.

**In a nutshell:** If you find yourself needing to coordinate multiple non-transactional resources, you should consider using compensating transactions.

# Code Example
In this code example, we have a simple service that is used by an ECommerce application to sell books.
As well as making updates to transactional resources (such as a database) it also needs to send an email notifying the customer that the order was made.

```
public class BookService {

    @Inject
    EmailSender emailSender;
 
    @Compensatable
    public void buyBook(String item, String emailAddress) {
 
        emailSender.notifyCustomerOfPurchase(item, emailAddress);
        //Carry out other activities, such as updating inventory and charging the customer
    }
}
```
The above class represents the BookService.
The `buyBook` method coordinates updates to the database and notifies the customer via an email.
The `buyBook` method is annotated with `@Compensatable`.
Processing of this annotation ensures that a compensation-based transaction is running when the method is invoked.
This annotation is processed similarly to the `@Transactional` annotation (new to JTA 1.2).
The key difference being that it works with a compensation-based transaction, rather than a JTA (ACID) transaction.
An uncaught `RuntimeException` (or subclass of) will cause the transaction to be canceled, and any completed work to be compensated.
Again, this behavior is based on the Transaction handling behavior of `@Transactional` in JTA 1.2.
For the sake of brevity, I have excluded the calls to update the other transactional resources.
Part 3 of this series will show interoperation with JTA ACID transactions.

```
public class EmailSender {

    @Inject
    OrderData orderData;
  
    @CompensateWith(NotifyCustomerOfCancellation.class)
    public void notifyCustomerOfPurchase(String item, String emailAddress) {
 
        orderData.setEmailAddress(emailAddress);
        orderData.setItem(item);
        //send email here...
    }
}
```

This class carries out the work required to notify the customer.
In this case it simulates the sending of an email.
The method `notifyCustomerOfPurchase` can later be compensated, should the transaction fail.
This is configured through the use of the `CompensateWith` annotation.
This annotation specifies which class to use to compensate the work done within the method.
For this compensation to be possible, it will need available to it, key information about the work completed.
In this case the item ordered and the address of the customer.
This data is stored in a CDI managed bean, `orderData`, which as we will see later, is also injected in the compensation handler.

```
@CompensationScoped
public class OrderData {

    private String item;
    private String emailAddress;
    ...
}
```

This managed bean represents the state required by the compensation handler to undo the work.
The key thing to notice here is that the bean is annotated with `@CompensationScoped`.
This scope is very similar to the `@TransactionScoped` annotation (new in JTA 1.2).
This annotation ensures that the lifecycle of the bean is tied to the current running transaction.
In this case the lifecycle of the compensation based transaction, but in the case of `@TransactionScoped` it is tied to the lifecycle of the JTA transaction.
The `@CompensationScoped` bean will also be serialized to the transaction log, so that it is available in the case that the compensation handler needs to be invoked at recovery time.

```
public class NotifyCustomerOfCancellation implements CompensationHandler {

    @Inject
    OrderData orderData;
 
    @Override
    public void compensate() {
        String emailMessage = "Sorry, your order for " + orderData.getItem() + " has been cancelled";
        //send email here...
    }
}
```

This class implements the compensation handler.
For our example it simply takes the details of the order from the injected OrderData bean and then sends an email to the customer informing them that the order failed.

# Summary
In this blog post I explained why it's difficult to coordinate non-transactional resources in an ACID transaction and showed how a compensation-based transaction can be used 
to solve this problem.
Part 3, of this series, will look at cross-domain distributed transactions: Here I'll show that ACID transactions are not always a good choice for scenarios where the transaction is distributed, and potentially crossing multiple business domains.
I'll show how a compensation-based transaction could be used to provide a better solution.