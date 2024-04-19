# Approximation pattern
- This pattern generates a statistically valid number that is not exact (like the total number of people in the world).
- This pattern is implemented on the application side.
- We use this pattern:
    - When data is difficult or expensive to calculate.
    - When getting the exact number is not critical.
- It's a good choice when working with Big Data.
- Advantages:
    - Reduces writes.
    - Can hel reduce contention on heavily updated documents (in some cases).

## Example
The example on the [Computed Pattern](../computed-pattern/) about the calculation of the reviews rating on every write, reduces the reading time but double the number of writes (because on every new review an update is performed on the book document). Note that in this case of the Computed Pattern example, the average of the rating reviews is accurate.

But, what if the accuracy is not important here? Or if we have tons of reviews? It really makes difference if we do not calculate the rating when the review number 1,000,001 was added?

In this cases, when our app sees that a book has already received a significant number of reviews, then it could decide to only recalculate the books reading periodically instead of on demand. It will drastically reduce the number of database writes by sacrificing some accuracy.

How to achieve this?
One way is to generate a random number between 1 and 10, in the app logic, when a new review is posted. In this case, the calculation of the book rating will be performed just when the random number is 10. So, we reduce the number of writes by 90%.

A important thing here is that we must change the formula to achieve this:
```
(avgRating * reviewCount + (newRating * 10)) / (reviewCount + 10)
```
