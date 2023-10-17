# New use case creation
## Kata goal

Implement a new use case that goes through all the hexagon

## Business

We now want to have booking created automatically and stored so that the next time we started the application it gets the already created bookings.

# Steps

## Add a creation use case 

Add a cron task that creates a new pallet booking in the `C:\data\bookings.csv` file 

- Prepare the inside part
  - Start with the test by using a kind of in memory persistence and without using mockito
  - Do the implementation
  - Write down your questions and choices
  - What was difficult or easy?
- Do the adapters
  - Is it easy to identify adapters?

### Code extracts

Here are some coding sample to faster your adapter creation

### File appender method

```java
    @Override
    public void create(PalletBooking palletBooking) {
        String bookingLine = palletBooking.getPartner() + "," + palletBooking.getQuantity() + System.getProperty("line.separator");
        try {
            Files.writeString(Path.of("c:\\data\\bookings.csv"), bookingLine, StandardOpenOption.APPEND);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

```
### Spring scheduler

```java
@Component
@EnableScheduling
public class BookingCreationScheduler {

    private final ForCreatingBooking forCreatingBooking;
    private final Random random = new Random();

    public BookingCreationScheduler() {
        ForHandlingBookings forGettingBooking = new FileBookingRepository();
        this.forCreatingBooking = new BookingManager(forGettingBooking);
    }

    //Every 15 seconds
    @Scheduled(fixedDelay = 15 * 1000)
    public void createBooking() {
        PalletBooking palletBooking = new PalletBooking("Cron partner", (long) random.nextInt(20));
        forCreatingBooking.create(palletBooking);
    }
}
```