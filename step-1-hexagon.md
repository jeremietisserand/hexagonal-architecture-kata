# Hexagonal architecture
## Kata goal

Implement the hexagonal architecture.

## Business

Booking pallets have been sent to the application.
We now want to compute the sum of all the exchanged pallets.

The two boundaries that we can already detect are the following:
- Provided use case: **sum the bookings**
- Needed to fulfill the request: **have all the bookings**

# Steps

## Implement the hexagon

### Prepare
- Create `booking` package (new package for booking management)
- Create `PalletBooking` class with two fields
  - `String partner`
  - `Long quantity`

## Application and port preparation

### Present the feature to perform (Driver side)
- Create the `booking.port.in` package
- Create the `ForCalculatingBookingSum` interface
- Add the method `Long computeSum()`

### Prepare the feature to code
- Create the `BookingManager` class in the `booking` package
- Add the `implements ForCalculatingBookingSum` to `BookingManager`
- Implement the method with a fake body

### Test the method
- Create the `BookingManagerTest`
- Create a first test named `sums`
- Test that the method `computeSum` of the `BookingManager` returns a specific value

Example:
```java
BookingManager bookingManager = new BookingManager();

Long sum = bookingManager.computeSum();

Assertions.assertThat(sum).isEqualTo(8L);
```

### Present the needed system (Driven side)
- Create the `booking.port.out` package
- Create the `ForGettingBookings` interface
- Add the method `Collection<PalletBooking> getBookings();`

### Use this needed system in the Manager
- Go in the `BookingManager`
- Create the field `private final ForGettingBookings bookingsRepository;`
- Create the constructor with the parameter
- Adapt the test to make it compile

### Make the test green
- Create a `ForGettingBookings` manual mock without Mockito (new Class in the test for instance)
- Make the mock return the values needed to make the test green
- Code the rule in the `BookingManager`

##  Adapters part: Communication with external systems

### Create the Driven adapter
- Create the `booking.adapter.out` package
- Create the `FileBookingRepository` that implements `ForGettingBookings`
- Implement the method that reads the file
- Create the csv file (here it will be located in `C:\data\bookings.csv`)


#### CSV content
```csv
FIRST PARTNER,5
SECOND PARTNER,8
THIRD PARTNER,10
```

#### Code Example
```java
    @Override
    public Collection<PalletBooking> getBookings() {
        return loadPalletBookings();
    }
  
    private Collection<PalletBooking> loadPalletBookings() throws RuntimeException {
        try {
            List<String> fileContent = Files.readAllLines(Path.of("c:\\data\\bookings.csv"));
            return fileContent.stream()
                    .map(line -> new PalletBooking(line.split(",")[0], Long.parseLong(line.split(",")[1])))
                    .collect(Collectors.toList());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

### Create the Driver adapter
- Create the `booking.adapter.in` package
- Create the `BookingRestController`
  - Add `@RestController` and `@RequestMapping("booking")` at the top of the file
  - Create a method that do the bookingSum

Example:
```java
    @GetMapping
    public ResponseEntity<Long> sumBookings() {
        return ResponseEntity.ok(forCalculatingBookingSum.computeSum());
    }
```
- Create the constructor without parameter 
- Instanciate manually the `BookingManager`

### Delegate the instanciation to spring
- Add the `@Service` on the top of the `BookingManager`
- Add the `@Repository` on the top of the `FileBookingRepository`
- Add the `BookingManager` parameter to the `BookingRestController`
