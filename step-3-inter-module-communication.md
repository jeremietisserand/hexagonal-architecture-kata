# Inter domain communication
## Kata goal

Two domains that need to communicate together.

We represent each domain in an hexagon in order to isolate them.

Implement the communication between two domains that are in the same deployment unit (modular monolith).

## Business

We want to verify the format of the business partner code.

There can be multiple reasons to have multiple hexagons:
- In the near future, we will have to implement more controls regarding business partners
- Dedicated team will be in charge of the 2nd hexagon
- 2nd hexagon will be replaced by an external system
- 2nd hexagon will be deployed as dedicated unit (a micro-service usable by other projects)

For now the format we want to check is:
- total length is **20 characters**
- example: **'FIHEL-partner 496871'**

At the end of the kata you will be able to check your code by using this URL in a browser: 
- `http://localhost:8080/booking/create/{businessPartnerCode}`
It should create with a booking with a quantity of 1 and return the sum of all the quantities 

# Steps

## Prepare
- Create `businesspartner` package (new package for booking management)

## Application and port preparation

### Present the feature to perform (Driver side)
- Create the `businesspartner.port.in` package
- Create the `ForCheckingBusinessPartnerCode` interface
- Add the method `boolean isValid(String code)`

### Prepare the feature to code
- Create the `BusinessPartnerVerifier` class in the `businesspartner` package
- Annotate this class with the `@Service` annotation
- Add the `implements ForCheckingBusinessPartnerCode` to `BusinessPartnerVerifier`
- Implement the method with a fake body

### Test and implement the rule
- Create the `BusinessPartnerVerifierTest`
- Create test methods (case true/false)
- Implement the rule in the `BusinessPartnerVerifier`

```java
class BusinessPartnerVerifierTest {

    private BusinessPartnerVerifier businessPartnerVerifier = new BusinessPartnerVerifier();

    @Test
    void isValid_returnsTrue_whenCodeIs20CharactersLong() {
        boolean result = businessPartnerVerifier.isValid("12345678901234567890");

        assertThat(result).isTrue();
    }

    @Test
    void isValid_returnsFalse_whenCodeIs5CharactersLong() {
        boolean result = businessPartnerVerifier.isValid("12345");

        assertThat(result).isFalse();
    }

}
```

### Prepare the call
- Create the `ForVerifyingBusinessPartner` interface in the `booking.port.out` package
- Add the method `boolean validate(String code)`
- Inject the `ForVerifyingBusinessPartner` in the `BookingManager`
- Add the call of the method `validate()` in the `BookingManager` and do not create booking if false
- In `BookingManagerTest` fix the compilation problem by creating a fake `ForVerifyingBusinessPartner` implementation that always returns true
- In `BookingRestController` inject the `ForVerifyingBusinessPartner` using constructor injection

##  Implement communication between systems

### Prepare the adapters

#### Create the booking `out` adapter:
- Create the `InMemoryBusinessPartnerVerifier` class
- Add the `implements ForVerifyingBusinessPartner` to `InMemoryBusinessPartnerVerifier`
- Override the method (with an empty body)
- Add the `@Repository` annotation to the class

#### Create the business partner `in` adapter:
- Create `businesspartner.adapter.in` package
- Create the `ProvideBusinessPartnerVerifier` class
- Inject `ForCheckingBusinessPartnerCode` to `ProvideBusinessPartnerVerifier`
- Create a  `public boolean validate(String code)` method
- Call the validate method from `ForCheckingBusinessPartnerCode`
- Add the `@Component` annotation to the class

### Implement adapters

Possible implementations:
- Point to point: `out` knows `in`
   - Direct in memory
- Publish / subscribe: `out` publishes, `in` subscribes
   - Inversion of Dependency

#### Tips

##### Direct in memory

In adapter `out`, inject the adapter `in`.

##### Inversion of Dependency

In adapter `out` create a new interface and implement it the adapter `in`

Inject the new interface in the `InMemoryBusinessPartnerVerifier` using Spring
