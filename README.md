# arrayconv

Class for holding methods that convert numpy arrays between binary representations and integer representations.
Doing this as a class speeds up the conversion functions by eliminating the need to recalculate the reversed exponential array, 
leading to a ~40% increase in speed.

### EXAMPLES
```
ac = ArrayConversion()
arr = np.array([1,2,4,8])
binarr = ac.int_to_bin(arr)
print(binarr)
```

[[0, 0, 0, 0, 0, 0, 0, 1],
 [0, 0, 0, 0, 0, 0, 1, 0],
 [0, 0, 0, 0, 0, 1, 0, 0],
 [0, 0, 0, 0, 1, 0, 0, 0]])   

```
intarr = ac.bin_to_int(binarr)
intarr
    [1 2 4 8]
```

### EXPLANATIONS
A critical component is the class variable 'revexp': '1 << np.arange(b))[::-1])'.
This is made by generating a increment integer array.
```
np.arange(8)

[0 1 2 3 4 5 6 7]
```

'[::-1]' is common notation and an easy way to reverse an array.
```
np.arange(8)[::-1]

[7 6 5 4 3 2 1 0]
```
'<<' is the bitshift operator for numpy arrays. Applying it leaves us with a reversed exponential array, thus: 'revexp'.
```
1 << np.arange(8)[::-1]

[128  64  32  16   8   4   2   1]
```


The int_to_bin function consists of a few different components. The first is the reshaping of the input 1D numpy array.
```
arr = np.array([1,2,4,8])
arr[:,None]

[[1]
 [2]
 [4]
 [8]]

```

'arr[:,None]' is functionally equivalent to the following, with the exception that for some reason it is faster.
>>> arr[:,np.newaxis]
>>> arr.reshape(arr.size,1)

Then, we use the '&' operator between the reshaped array, and the revexp array. Because of a mechanism called 'broadcasting' in numpy,
we observe the following behavior.
>>> A = np.array([[1],
                    [0],
                    [1]])
>>> B = np.array([1,1,1])
>>> A * B
    [[1 1 1]
        [0 0 0]
        [1 1 1]]

This behavior manifests because the first element in A [1] is multiplied by each element in B [1,1,1], 
the second element in A [0] is multiplied by each element in B [1,1,1], etc.
The same behavior is true for the logical_and or '&' operator. Notably, the & operator can appear confusing
at first, because when applied on non-binary ints, it behaves in the following way:
    7 & 3 = 3
    7 & 13 = 5
    8 & 3 = 0
    8 & 13 = 8

The reason it does is, is that it implicitly treats each number as a binary number. Then, it does an elementwise logical_and
operation on the two binary numbers. It is easier to see that this makes sense by taking the above examples and
displaying them in binary (as four bit numbers):
    [0 1 1 1] & [0 0 1 1] = [0 0 1 1]
    [0 1 1 1] & [1 1 0 1] = [0 1 0 1]
    [1 0 0 0] & [0 0 1 1] = [0 0 0 0]
    [1 0 0 0] & [1 1 0 1] = [1 0 0 0]

Thus, the logical and between the parameter array of ints, and the revexp array of ints is broadcasted, and the and operation is performed.

    



SOURCES
    The source for the original method of convering integer arrays to binary can be found here:
    https://stackoverflow.com/questions/22227595/convert-integer-to-binary-array-with-suitable-padding