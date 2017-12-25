+++
date = "2015-08-29T18:43:23-05:00"
draft = true
title = "Duplicated code"

+++
Number one in the stink parade is duplicated code. If you see the same code structure in more than one place, you can be sure that your program will be better if you find a way to unify them.

When coding I focus mostly on duplication. *When the same thing is done over and over, its a sign that there is an idea in our mind that is no well represented in the code. I try to figure out what it is, then I try to express that idea more clearly.* ~ Ron Jeffries

* When you have the same expression in two methods of the same class. (Extract method)
* When you have the same expression in two sibling subclasses. (Pull up method)
* If the code is similar but not the same. (Form template method)
* If the methods do the same thing with different algorithm. (Substitute Algorithm)
* If you have duplicated code in two unrelated classes. (Extract class)

I will show you an example of the first case (Extract Method). Consider the following snippet of code.

```java
@Test
public void shouldFullAParkingLot() throws Exception {
  assertFalse(parkingLot.isFull());
  for(inti=0;i<MAX_CAPACITY;i++){
    parkingLot.park();
  }
}

@Test(expected=ParkingLotOverFlowException.class)
public void shouldThrowAnExceptionSinceIsFullAndWantToPark throws Exception {
  assertFalse(parkingLot.isFull());
  for(inti=0;i<MAX_CAPACITY;i++){
    parkingLot.park();
  }
  parkingLot.park();
}
```

As you can see the code is almost identical in the two tests, although they test different scenarios, so we applied extract method in order to avoid duplication.

```java
@Test
public void shouldFullAParkingLot() throws Exception {
  fullMyParkingLot();
}

@Test(expected=ParkingLotOverFlowException.class)
public void shouldThrowAnExceptionSinceIsFullAndWantToPark throws Exception {
  fullMyParkingLot();
  parkingLot.park();
}

private void fullMyParkingLot() throws ParkingLotOverFlowException {
  assertFalse(parkingLot.isFull());
  for(inti=0;i<MAX_CAPACITY;i++){
    parkingLot.park();
  }
  assertTrue(parkingLot.isFull());
}
```

*The biggest problem with extract method is dealing with local variables, and temps are one of the main resources of this issue.* ~ Martin Fowler.

To download the project:

```bash
git clone https://github.com/josdem/refactoring.git
git fetch
git checkout feature/duplicated-code-setup
git checkout feature/duplicated-code-complete
```

To run the project:

```bash
gradle test
```

Read more, returning to the main article. [Refactoring](/techtalk/refactoring)
