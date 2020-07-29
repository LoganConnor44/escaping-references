# Java

##  Escaping References

### What Does Escaping Reference Mean?

An escaping reference is when an object is unintentionally exposing mutability to developers.

### Why Is This Bad?

* Developers may, accidentally, change the data in a class field through an exposed getter property.
* This can be problematic for concurent processes.

### What Is An Example Of This?

Imagine a `CustomerRecords` that has getter and setter properties. One of the getters defined is `getCustomers()`.

```Java
public class CustomerRecords {
    private Map<String, Customer> records;
    
    public CustomerRecords() {
        this.records = new HashMap<>();
    }

    public void addCustomer(Customer customer) {
        this.records.put(customer.getName(), customer);
    }

    public Map<String, Customer> getCustomers() {
        return this.records;
    }
}
```

As it stands, if a developer would like to read the `records` field, they would call `getCustomers()`. It contains a reference to the `records` collection. The problem here being that the `records` data is mutable. There is nothing to stop the developer from calling `getCustomers().clear()` to wipe the data. An example of this is as followed:

```Java
CustomerRecords customerRecords = new CustomerRecords();
customerRecords.addCustomer(new Customer("John"));

Map<String, Customer> myCustomers = customerRecords.getCustomers();
myCustomers.clear();
```

This is deviating from the intention from the original developer's purpose of `getCustomers()`, which is to iterate over the collection. It's almost as if we have declared this field as public and have now violated OOP encapusulation rules. This will create a difficult task of debugging the source of invalid data in the records collection, should one occur, because any code in our application can now access *and* mutate this collection.

### How Do We Prevent This?

#### First Example

To reiterate the problem, it is important to mention that references can only escape if we have methods that are returning a pointer to an object.

There are a few ways (with their own pitfalls) to avoid this, but the best way is to return a read-only copy of the object. Let's change the `getCustomers()` property to return an unmodifiable map. 

```Java
public Map<String, Customer> getCustomers() {
    return Collections.unmodifiableMap(this.records);
}
```

If a developer now attempts to mutate the collection an `UnsupportedOperationException` will be thrown as runtime.

#### Second Example

Again, it is important to return a read-only object so we do not create an escaping references.

Let's now look at returning a `Customer` object from the `CustomerRecords` class. We will add a method to do this within `CustomerRecords`.

```Java
public Customer getCustomerByName(String name) {
    return this.records.get(name);
}
```

Here, we have actually created another escaping reference since the returned `Customer` object may now be modified. The solution is to return an interface, rather than the class. Let's take a look at our `Customer` class then write an interface that will function as a read-only version of the class and lastly override the get method to behave properly.

```Java
public Customer {
    public Customer() {
        private String name;

        public Customer(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```

Here is our `Customer` class. Now, to make this effectively read-only, let's have it implement an interface that our `CustomerRecords` will return.

```Java
public interface ICustomerReadOnly {
    public abstract String getName();
}
```

Update our `Customer` class to implement this interface.

```Java
public Customer implements ICustomerReadOnly {
    public Customer() {
        private String name;

        public Customer(String name) {
            this.name = name;
        }

        @Override
        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```

Let's update our `CustomerRecords` class to reflect this as well.

```Java
public ICustomerReadOnly getCustomerByName(String name) {
    return this.records.get(name);
}
```

When a developer calls `getCustomerByName()` they will only be able to see the `getName()` method. Now, this isn't completely fool-proof. This could be casted to a `Customer` class, but for this purpose this is enough of a safe-guard.
