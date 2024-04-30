# Anti Patterns

## Index
- [Unbounded arrays](#unbounded-arrays)
- [Bloated Documents](#bloated-documents)
- [Massiver Number of collections](#massive-number-of-collections)
- [Unnecessary indexes](#unnecessary-indexes)
- [Data Normalization](#data-normalization)
- [Case-Sensitivity](#case-sensitivity)

### Unbounded arrays
- An unbounded array is a large, growing array with an unlimited number of elements.
- What to avoid it?
    - We have the risk to exceed the BSON document size limit of 16MB.
    - We can also have a decrease in the index performance.
- What to consider:
    - Only store data together if it is queried together.
    - An array should not grow without bound.
    - High cardinality arrays should not be embedded.
- Design patterns that can help us to solve this: __Extended Reference__ and the __Subset__ patterns.

### Bloated Documents
To do.

### Massive Number of Collections
To do.

### Unnecessary Indexes
To do.

### Data Normalization
To do.

### Case-Sensitivity
To do.
