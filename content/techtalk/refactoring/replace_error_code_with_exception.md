+++
date = "2015-08-29T19:51:20-05:00"
draft = true
title = "Replace Error Code with Exception"

+++
A method returns special code to indicate a wrong path. Throw an exception instead.

In computers, as in life, things go wrong occasionally, when things go wrong, you need to do something about it. In the simplest case, you can stop the program with an error code. When a routine finds an error, it needs to let its caller know, and the caller may pass the error up to the chain.

## Example
Let’s consider the following class who try to withdraw an amount from user, and return true as success or false as failure at attemp.

```java
package com.josdem.refactoring;
import java.math.BigDecimal;

public class TransactionApplier {

  private AmountValidator amountValidator = new AmountValidator();

  public Boolean subtractAmount(User user, BigDecimal amount) {
    if(amountValidator.hasFunds(user, amount)){
      user.setBalance(user.getBalance().subtract(amount));
      return true;
    }
    return false;
  }

}
```

TransactionApplier has a collaborator, AmountValidator who knows if user has funds or not. Here the code.

```java
package com.josdem.refactoring;
import java.math.BigDecimal;

public class AmountValidator {

  public Boolean hasFunds(User user, BigDecimal amount) {
    return user.getBalance().compareTo(amount) >= 0;
  }

}
```

Now refactor, first we change AmountValidator to throw an exception if user has no funds

```java
package com.josdem.refactoring;
import java.math.BigDecimal;

public class AmountValidator {

  public void hasFunds(User user, BigDecimal amount) {
    if(user.getBalance().compareTo(amount) < 0){
      throw new RuntimeException("No Sufficient Funds");
    }
  }

}
```

Now we are ready to delete boolean code from TransactionApplier

```java
package com.josdem.refactoring;
import java.math.BigDecimal;

public class TransactionApplier {

  private AmountValidator amountValidator = new AmountValidator();

  public void subtractAmount(User user, BigDecimal amount) {
    amountValidator.hasFunds(user, amount);
    user.setBalance(user.getBalance().subtract(amount));
  }

}
```

This is the test case to cover functionality

```java
package com.josdem.refactoring;

import static org.junit.Assert.assertEquals;
import java.math.BigDecimal;
import org.junit.Test;

public class TestTransactionApplier {
  
  private TransactionApplier transactionApplier = new TransactionApplier();

  private BigDecimal userBalance = new BigDecimal(100);

  @Test
  public void shouldSubstractAmount() {
    BigDecimal amount = new BigDecimal(20);
    BigDecimal expectedBalance = new BigDecimal(80);
    
    User user = setUserExpectations();
    
    transactionApplier.subtractAmount(user, amount);
    assertEquals(expectedBalance, user.getBalance());
  }
  
  @Test (expected=RuntimeException.class)
  public void shouldNotSubtractAmountDueToNotSufficientFunds() {
    BigDecimal amount = new BigDecimal(200);
    User user = setUserExpectations();
    
    transactionApplier.subtractAmount(user, amount);
    
    assertEquals(userBalance, user.getBalance());
  }

  private User setUserExpectations() {
    User user = new User();
    user.setBalance(userBalance);
    return user;
  }

}
```

To download the project:

```bash
git clone https://github.com/josdem/refactoring.git
git fetch
git checkout feature/replace-error-code-with-exception-setup
git checkout feature/replace-error-code-with-exception-complete
```

To run the project:

```bash
gradle test
```

Return to the main article. [Refactoring](/techtalk/refactoring)
