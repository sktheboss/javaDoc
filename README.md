## First start with the constructor of ArrayList

** ArrayList has three ways to initialize, the source code of the construction method is as follows: **

`` `java
   / **
     * Default initial capacity
     * /
    private static final int DEFAULT_CAPACITY = 10;
    

    private static final Object [] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    / **
     * The default constructor, constructs an empty list with an initial capacity of 10 (constructed without parameters)
     * /
    public ArrayList () {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
    / **
     * Constructor with initial capacity parameters. (User-specified capacity)
     * /
    public ArrayList (int initialCapacity) {
        if (initialCapacity> 0) {// Initial capacity is greater than 0
            // Create an array of initialCapacity size
            this.elementData = new Object [initialCapacity];
        } else if (initialCapacity == 0) {// The initial capacity is equal to 0
            // Create empty array
            this.elementData = EMPTY_ELEMENTDATA;
        } else {// Initial capacity is less than 0, throw an exception
            throw new IllegalArgumentException ("Illegal Capacity:" +
                                               initialCapacity);
        }
    }


   / **
    * Constructs a list containing the elements of the specified collection, which are returned in order using the iterator of the collection
    * If the specified collection is null, throws NullPointerException.
    * /
     public ArrayList (Collection <? extends E> c) {
        elementData = c.toArray ();
        if ((size = elementData.length)! = 0) {
            // c.toArray might (incorrectly) not return Object [] (see 6260652)
            if (elementData.getClass ()! = Object []. class)
                elementData = Arrays.copyOf (elementData, size, Object []. class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

`` `

Attentive classmates will surely find that: ** When creating an ArrayList with a parameterless construction method, an empty array is actually initialized and assigned. When the array is actually added, the capacity is actually allocated. That is, when the first element is added to the array, the array capacity is expanded to 10. ** We will talk about this when we analyze the expansion of ArrayList!

## Two Step by Step Analysis of ArrayList Expansion Mechanism

Here is an example of an ArrayList created by a parameterless constructor

### 1. Look at the add method first

`` `java
    / **
     * Appends the specified element to the end of this list.
     * /
    public boolean add (E e) {
   // Before adding elements, first call ensureCapacityInternal method
        ensureCapacityInternal (size + 1); // Increments modCount !!
        // Seeing the essence of adding elements to ArrayList is equivalent to assigning values ​​to the array
        elementData [size ++] = e;
        return true;
    }
`` `
### 2. Take a look at the `ensureCapacityInternal ()` method

You can see that the add method first called `ensureCapacityInternal (size + 1)`

`` `java
   // Get the minimum expansion capacity
    private void ensureCapacityInternal (int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // Get the larger value of the default capacity and incoming parameters
            minCapacity = Math.max (DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity (minCapacity);
    }
`` `
** When adding the first element, the minCapacity is 1, and after the Math.max () method comparison, the minCapacity is 10. **

### 3. `ensureExplicitCapacity ()` method

If you call the `ensureCapacityInternal ()` method, you will definitely enter (execute) this method. Let's study the source code of this method!

`` `java
  // Judge whether you need to expand
    private void ensureExplicitCapacity (int minCapacity) {
        modCount ++;

        // overflow-conscious code
        if (minCapacity-elementData.length> 0)
            // Call the grow method for capacity expansion. Calling this method indicates that capacity expansion has begun.
            grow (minCapacity);
    }

`` `

Let's analyze it carefully:

-When we add the first element to the ArrayList, elementData.length is 0 (because it is still an empty list), because the `ensureCapacityInternal ()` method is executed, so minCapacity is 10 at this time. At this point, `minCapacity-elementData.length> 0` holds, so it will enter `grow (minCapacity)` method.
-When adding the second element, the minCapacity is 2. At this time, elementData.length (capacity) is expanded to 10 after adding the first element. At this point, `minCapacity-elementData.length> 0` does not hold, so it will not enter (execute) the `grow (minCapacity)` method.
-When adding the 3rd, 4th ... to the 10th element, the grow method is still not executed, and the array capacity is 10.

Until the eleventh element is added, minCapacity (11) is larger than elementData.length (10). Enter the grow method for capacity expansion.

### 4. `grow ()` method

`` `java
    / **
     * Maximum array size to allocate
     * /
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE-8;

    / **
     * The core method of ArrayList expansion.
     * /
    private void grow (int minCapacity) {
        // oldCapacity is the old capacity, newCapacity is the new capacity
        int oldCapacity = elementData.length;
        // Shift oldCapacity to the right by one bit, the effect is equivalent to oldCapacity / 2,
        // We know that the bit operation is much faster than the division operation. The result of the entire sentence expression is to update the new capacity to 1.5 times the old capacity.
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // Then check if the new capacity is greater than the minimum required capacity. If it is still less than the minimum required capacity, then consider the minimum required capacity as the new capacity of the array.
        if (newCapacity-minCapacity <0)
            newCapacity = minCapacity;
       // If the new capacity is greater than MAX_ARRAY_SIZE, enter (execute) the `hugeCapacity ()` method to compare minCapacity and MAX_ARRAY_SIZE,
       // If minCapacity is greater than the maximum capacity, the new capacity is `Integer.MAX_VALUE`; otherwise, the new capacity is MAX_ARRAY_SIZE which is` Integer.MAX_VALUE-8`.
        if (newCapacity-MAX_ARRAY_SIZE> 0)
            newCapacity = hugeCapacity (minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf (elementData, newCapacity);
    }
`` `

** int newCapacity = oldCapacity + (oldCapacity >> 1), so the capacity of the ArrayList will be 1.5 times the original after each expansion! (After JDK1.6) ** For JDk1.6, the capacity after expansion is 1.5 times +1! Please refer to the source code for details

> ">>" (shift operator): >> 1 Shifting one bit to the right is equivalent to dividing 2 and shifting n bits to the right is equivalent to dividing to the power of n. Here oldCapacity is obviously shifted to the right by 1 bit so it is equivalent to oldCapacity / 2. For binary operation of big data, the shift operator is much faster than those of ordinary operators, because the program just moves it and does not calculate, which improves efficiency and saves resources

** Let's explore the `grow ()` method by example again: **

-When adding the first element, oldCapacity is 0, and the first if judged after comparison is true, newCapacity = minCapacity (10). But the second if judgement will not hold, that is, newCapacity is not larger than MAX_ARRAY_SIZE, then it will not enter the `hugeCapacity` method. The size of the array is 10. In the add method, return true and the size increase to 1.
-When the eleventh element of add enters the grow method, newCapacity is 15, which is greater than minCapacity (which is 11). The new capacity is not greater than the maximum size of the array and will not enter the hugeCapacity method. The size of the array is expanded to 15, the return method in the add method is increased, and the size is increased to 11.
-And so on ...

** It is important to add here, but it is easy to overlook:

-The `length` property in java is for arrays.For example, if you declare an array, you want to know the length of the array using the length property.
-The length () method in java is for strings.If you want to see the length of the string, use the length () method.
-The `size ()` method in java is for generic collections. If you want to see how many elements there are in this generic, call this method to see!

### 5. `hugeCapacity ()` method.

From the source of the `grow ()` method above, we know: If the new capacity is greater than MAX_ARRAY_SIZE, enter (execute) the `hugeCapacity ()` method to compare minCapacity and MAX_ARRAY_SIZE. If minCapacity is greater than the maximum capacity, the new capacity is `Integer.MAX_VALUE` , Otherwise, the new capacity is MAX_ARRAY_SIZE which is `Integer.MAX_VALUE-8`.


`` `java
    private static int hugeCapacity (int minCapacity) {
        if (minCapacity <0) // overflow
            throw new OutOfMemoryError ();
        // Compare minCapacity and MAX_ARRAY_SIZE
        // If minCapacity is large, use Integer.MAX_VALUE as the size of the new array
        // If MAX_ARRAY_SIZE is large, use MAX_ARRAY_SIZE as the size of the new array
        // MAX_ARRAY_SIZE = Integer.MAX_VALUE-8;
        return (minCapacity> MAX_ARRAY_SIZE)?
            Integer.MAX_VALUE:
            MAX_ARRAY_SIZE;
    }
`` `



## Three `System.arraycopy ()` and `Arrays.copyOf ()` methods


Reading the source code, we will find that these two methods are called a lot in ArrayList. For example: the expansion operation we mentioned above and methods like `add (int index, E element)` and `toArray ()` all use this method!


### 3.1 `System.arraycopy ()` method

`` `java
    / **
     * Inserts the specified element at the specified position in this list.
     * First call rangeCheckForAdd to check the bounds of index; then call ensureCapacityInternal method to ensure that the capacity is large enough;
     * Then move all the members from the index back one position; insert the element into the index position; finally add 1 to the size.
     * /
    public void add (int index, E element) {
        rangeCheckForAdd (index);

        ensureCapacityInternal (size + 1); // Increments modCount !!
        // arraycopy () method copies the array itself
        // elementData: source array; index: starting position in source array; elementData: target array; index + 1: starting position in target array; size-index: number of array elements to be copied;
        System.arraycopy (elementData, index, elementData, index + 1, size-index);
        elementData [index] = element;
        size ++;
    }
`` `

We write a simple method to test the following:

`` `java
public class ArraycopyTest {

public static void main (String [] args) {
// TODO Auto-generated method stub
int [] a = new int [10];
a [0] = 0;
a [1] = 1;
a [2] = 2;
a [3] = 3;
System.arraycopy (a, 2, a, 3, 3);
a [2] = 99;
for (int i = 0; i <a.length; i ++) {
System.out.println (a [i]);
}
}

}
`` `

result:

`` `
0 1 99 2 3 0 0 0 0 0
`` `

### 3.2 `Arrays.copyOf ()` method

`` `java
   / **
     Returns an array containing all the elements in this list (from the first to the last element) in the correct order; the runtime type of the returned array is the runtime type of the specified array.
     * /
    public Object [] toArray () {
    // elementData: the array to be copied; size: the length to be copied
        return Arrays.copyOf (elementData, size);
    }
`` `

Personally, I think the use of the `Arrays.copyOf ()` method is mainly to expand the original array. The test code is as follows:

`` `java
public class ArrayscopyOfTest {

public static void main (String [] args) {
int [] a = new int [3];
a [0] = 0;
a [1] = 1;
a [2] = 2;
int [] b = Arrays.copyOf (a, 10);
System.out.println ("b.length" + b.length);
}
}
`` `

result:

`` `
10
`` `

### 3.3 The connection and difference between the two

** Contact: **

Looking at the source code of both, you can find that copyOf () actually calls the `System.arraycopy ()` method.

**the difference:**

`arraycopy ()` requires the target array. Copy the original array into your own defined array or the original array, and you can choose the starting point and length of the copy and the position in the new array. Create a new array and return it.

## Four `ensureCapacity` methods

There is an `ensureCapacity` method in the ArrayList source code. I don't know if you noticed it. This method ArrayList has not been called internally, so it is obviously provided for users to call. So what is the function of this method?

`` `java
    / **
    If necessary, increase the capacity of this ArrayList instance to ensure that it can hold at least the number of elements specified by the minimum capacity parameter.
     *
     * @param minCapacity required minimum capacity
     * /
    public void ensureCapacity (int minCapacity) {
        int minExpand = (elementData! = DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity> minExpand) {
            ensureExplicitCapacity (minCapacity);
        }
    }

`` `

** It is best to use the `ensureCapacity` method before adding a large number of elements to reduce the number of incremental reallocations **

We actually test the effect of the following method with the following code:

`` `java
public class EnsureCapacityTest {
public static void main (String [] args) {
ArrayList <Object> list = new ArrayList <Object> ();
final int N = 10000000;
long startTime = System.currentTimeMillis ();
for (int i = 0; i <N; i ++) {
list.add (i);
}
long endTime = System.currentTimeMillis ();
System.out.println ("Before using the ensureCapacity method:" + (endTime-startTime));

}
}
`` `

operation result:

`` `
Before using ensureCapacity method: 2158
`` `

`` `java
public class EnsureCapacityTest {
    public static void main (String [] args) {
        ArrayList <Object> list = new ArrayList <Object> ();
        final int N = 10000000;
        list = new ArrayList <Object> ();
        long startTime1 = System.currentTimeMillis ();
        list.ensureCapacity (N);
        for (int i = 0; i <N; i ++) {
            list.add (i);
        }
        long endTime1 = System.currentTimeMillis ();
        System.out.println ("After using ensureCapacity method:" + (endTime1-startTime1));
    }
}
`` `

operation result:

`` `

Before using the ensureCapacity method: 1773
`` `

From the results, we can see that it is best to use the `ensureCapacity` method before adding a large number of elements to the ArrayList to reduce the number of incremental reallocations.
