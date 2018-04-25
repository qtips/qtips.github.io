---
layout: post
title: Automatic save of managed JPA entities outside of transaction
enabledisqus: true
---

Repositories and transactions in Spring go hand in hand. All database access in Spring should be run inside a transaction, and you typically have `@Transactional` somewhere to enforce this. However, this is not always necessary. For example, when using Spring Data your repositories use `SimpleJPARepository` for CRUD functionality. The `SimpleJPARepository` uses `@Transactional` so when you perform CRUD operations, transactions are already handled for you. This can give a wrong impression that you do not need to annotate your own classes with `@Transactional`, because this is only true if you know what you are doing. 

Consider the following Spring Data based timeseries example for managing car rentals: 
{% highlight java %}
public CarRentalEntry createNewRental(Car car) {
  CarRentalEntry latestEntry = carRentalRepository.findByCarId(car.getId());
  latestCarRentalEntry.setEndDate(LocalDate.now());
  
  CarRentalEntry newEntry = new CarRentalEntry();
  newEntry.setCarId(car.getId())
  newEntry.setStartDate(LocalDate.now());
  newEntry.setEndDate(null);
  carRentalRepository.save(newEntry);
}  
{% endhighlight %}
In the example above, the latest car rental entry for a particular car is fetched through the repository and ended. Then a new car rental entry  is created and saved. This will work without `@Transactional` because `carRentalRepository` is a `SimpleJPARepository` which handles the transactions. Now consider the following where the saving is done before changing the end date of `latestEntry`:  
 
 {% highlight java %}
 public CarRentalEntry createNewRental(Car car) { 
   CarRentalEntry newEntry = new CarRentalEntry();
   newEntry.setCarId(car.getId())
   newEntry.setStartDate(LocalDate.now());
   newEntry.setEndDate(null);
   carRentalRepository.save(newEntry);
   
   CarRentalEntry latestEntry = carRentalRepository.findByCarId(car.getId());
   latestCarRentalEntry.setEndDate(LocalDate.now());
 }  
 {% endhighlight %}

Functionally, the method is exactly the same, but in this example **only the save will be performed**. Modification of `latestEntry` will not be saved to the database because there is no transaction! To make this approach work `createNewRental()` must be annotated with `@Transactional`. 
Any changes on a JPA managed entity are only automatically saved if they happen inside a transaction which is normal JPA behaviour. So the question is WHY did the first approach not require a transaction. 

Actually it did. When `latestEntry` was fetches through the repository, it was added to the `persistanceContext` (a.k.a. Level 1 Cache) of JPAs `entityManager`. When the `save()` method was called, it flushed the `persistanceContext` upon transaction commit, which in turn had the side effect of also persisting the modified `latestEntry`. In the second example, `persistanceContext` did not have `latestEntry` upon calling `save()`. Because there is no transaction that commits when the methods completes, the changes are not flushed. By adding `@Transactional`, the `persistanceContext` is again flushed and the modification is written to the database. Note that the second example would also have worked without `@Transactional` by calling `carRentalRepository.flush()` because it too runs under a `@Transactional`. 

The bottom line is that you should have control of your own transactions because as this case shows it is easy to make mistakes.

Lastly a tip when debugging Hibernate and managed entity issues. Good candidate classes to place a break point are: 

* `org.springframework.orm.jpa.JpaTransactionManager`
* `org.hibernate.jpa.internal.TransactionImpl.commit()` persistence context that is to be flushed is typically found in `TransactionImpl.entityManager.session.persistenceContext`  