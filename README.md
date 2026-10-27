import java.util.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
class Seat {
    String seatNumber;
    boolean booked;
    Seat(String seatNumber) {
        this.seatNumber = seatNumber;
        this.booked = false;
    }
}
class Aircraft {
    String aircraftNumber;
    Map<String, Seat> seats = new LinkedHashMap<>();
    Aircraft(String aircraftNumber, int totalSeats) {
        this.aircraftNumber = aircraftNumber;
        for (int i = 1; i <= totalSeats; i++) {
            int row = (i - 1) / 6 + 1;
            char letter = (char) ('A' + (i - 1) % 6);
            String seatNo = row + "" + letter;
            seats.put(seatNo, new Seat(seatNo));
        }
    }
    int availableSeats() {
        int count = 0;
        for (Seat seat : seats.values()) {
            if (!seat.booked) count++;
        }
        return count;
    }
}
class Flight {
    String flightNumber, source, destination, date, time;
    double baseFare;
    Aircraft aircraft;
    Flight(String flightNumber, String source, String destination,
           String date, String time, double baseFare,int totalSeats) {
        this.flightNumber = flightNumber;
        this.source = source;
        this.destination = destination;
        this.date = date;
        this.time = time;
        this.baseFare = baseFare;
        this.aircraft = new Aircraft(flightNumber, totalSeats);
    }
}
class Passenger {
    int passengerId;
    String name, email, phone;
    Passenger(int passengerId, String name, String email, String phone) {
        this.passengerId = passengerId;
        this.name = name;
        this.email = email;
        this.phone = phone;
    }
}
class Payment {
    String paymentMethod, transactionId;
    double amount;
    boolean paid;
    Payment(double amount, String paymentMethod) {
        this.amount = amount;
        this.paymentMethod = paymentMethod;
        this.transactionId = "TXN" + System.currentTimeMillis();
        this.paid = true;
    }
}
class Booking {
    String bookingId;
    Flight flight;
    Passenger passenger;
    List<Seat> seats = new ArrayList<>();
    Payment payment;
    boolean cancelled;
    boolean checkedIn;
    Booking(String bookingId, Flight flight, Passenger passenger) {
        this.bookingId = bookingId;
        this.flight = flight;
        this.passenger = passenger;
        this.cancelled = false;
        this.checkedIn = false;
    }
    double getFare() {
        return flight.baseFare * seats.size();
    }
    double getTax() {
        return getFare() * 0.05;
    }
    double getTotal() {
        return getFare() + getTax();
    }
}
class BoardingPass {
    Booking booking;
    String gate;
    String timestamp;
    BoardingPass(Booking booking) {
        this.booking = booking;
        this.gate = "G" + (new Random().nextInt(10) + 1);
        this.timestamp = LocalDateTime.now().format(
            DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm:ss"));
    }
    void display() {
        System.out.println("\n========== BOARDING PASS ==========");
        System.out.println("Passenger : " + booking.passenger.name);
        System.out.println("Flight    : " + booking.flight.flightNumber);
        System.out.println("From      : " + booking.flight.source);
        System.out.println("To        : " + booking.flight.destination);
        System.out.println("Date      : " + booking.flight.date);
        System.out.println("Time      : " + booking.flight.time);
        for (Seat seat : booking.seats) {
            System.out.println("Seat      : " + seat.seatNumber);
        }
        System.out.println("Gate      : " + gate);
        System.out.println("Booking ID: " + booking.bookingId);
        System.out.println("Issued At : " + timestamp);
        System.out.println("===================================");
    }
}
public class AirlineReservationSystem {
    static Scanner sc = new Scanner(System.in);
    static Map<String, Flight> flights = new LinkedHashMap<>();
    static Map<Integer, Passenger> passengers = new LinkedHashMap<>();
    static Map<String, Booking> bookings = new LinkedHashMap<>();
    static int passengerCounter = 1;
    static int bookingCounter = 1001;
    static String read(String message) {
        System.out.print(message);
        return sc.nextLine().trim();
    }
    static int readInt(String message) {
        while (true) {
            try {
                return Integer.parseInt(read(message));
            } catch (NumberFormatException e) {
                System.out.println("Enter a valid number.");
            }
        }
    }
    static double readDouble(String message) {
        while (true) {
            try {
                double value = Double.parseDouble(read(message));
                if (value > 0) return value;
            } catch (NumberFormatException e) {
            }
            System.out.println("Enter a valid positive amount.");
        }
    }
    static void addFlight() {
        System.out.println("\n--- ADD FLIGHT ---");
        String number = read("Flight Number: ").toUpperCase();
        if (flights.containsKey(number)) {
            System.out.println("Flight already exists!");
            return;
        }
        String from = read("From: ");
        String to = read("To: ");
        String date = read("Departure Date (YYYY-MM-DD): ");
        String time = read("Departure Time (HH:MM): ");
        int totalSeats = readInt("Total Seats (maximum 120): ");
        if (totalSeats < 1 || totalSeats > 120) {
            System.out.println("Seat count must be between 1 and 120.");
            return;
        }
        double fare = readDouble("Base Fare per Seat: ");
        Flight flight = new Flight(number, from, to, date, time, fare, totalSeats);                             
        flights.put(number, flight);
        System.out.println("Flight added successfully!");
    }
    static void addPassenger() {
        System.out.println("\n--- ADD PASSENGER ---");
        String name = read("Passenger Name: ");
        if (name.isEmpty()) {
            System.out.println("Name cannot be empty.");
            return;
        }      
  String email = read("Email: ");
        String phone = read("Phone Number (10 digits): ");
        if (!phone.matches("[0-9]{10}")) {
            System.out.println("Invalid phone number!");
            return;
        }
        Passenger passenger = new Passenger(passengerCounter++, name, email, phone);
        passengers.put(passenger.passengerId, passenger);
        System.out.println("Passenger added successfully!");
        System.out.println("Passenger ID: " + passenger.passengerId);
    }
    static void displayReceipt(Booking b) {
        System.out.println("\n========== BOOKING RECEIPT ==========");
        System.out.println("Booking ID : " + b.bookingId);
        System.out.println("Passenger  : " + b.passenger.name);
        System.out.println("Flight     : " + b.flight.flightNumber);
        System.out.println("From       : " + b.flight.source);
        System.out.println("To         : " + b.flight.destination);
        System.out.println("Date       : " + b.flight.date);
        System.out.println("Time       : " + b.flight.time);
        System.out.print("Seats      : ");
        for (Seat seat : b.seats) {
            System.out.print(seat.seatNumber + " ");
        }
        System.out.printf("%nFare       : Rs. %.2f%n", b.getFare());
        System.out.printf("Taxes (5%%) : Rs. %.2f%n", b.getTax());
        System.out.printf("Total      : Rs. %.2f%n", b.getTotal());
        System.out.println("======================================");
    }
    static void createBooking() {
        System.out.println("\n--- CREATE BOOKING ---");
        if (flights.isEmpty() || passengers.isEmpty()) {
            System.out.println("Add a flight and passenger first.");
            return;
        }
        displayFlights();
        String flightNo = read("Enter Flight Number: ").toUpperCase();
        Flight flight = flights.get(flightNo);
        if (flight == null) {
            System.out.println("Flight not found!");
            return;
        }
        int passengerId = readInt("Enter Passenger ID: ");
        Passenger passenger = passengers.get(passengerId);
        if (passenger == null) {
            System.out.println("Passenger not found!");
            return;
        }
        System.out.println("Available Seats:");
        for (Seat seat : flight.aircraft.seats.values()) {
            if (!seat.booked) {
                System.out.print(seat.seatNumber + " ");
            }
        }
        System.out.println();
        int count = readInt("Number of Seats: ");
        if (count < 1 || count > flight.aircraft.availableSeats()) {
            System.out.println("Invalid number of seats.");
            return;
        }
        String[] requested = read(
            "Enter seat numbers separated by commas: ")
            .toUpperCase().replace(" ", "").split(",");
        if (requested.length != count) {
            System.out.println("Seat count does not match selection.");
            return;
        }
        Set<String> uniqueSeats = new HashSet<>();
        List<Seat> selectedSeats = new ArrayList<>();
        for (String seatNo : requested) {
            Seat seat = flight.aircraft.seats.get(seatNo);
            if (seat == null || seat.booked || !uniqueSeats.add(seatNo)) {
                System.out.println(
                    "Invalid, unavailable, or duplicate seat: " + seatNo);
                return;
            }
            selectedSeats.add(seat);
        }
        for (Seat seat : selectedSeats) {
            seat.booked = true;
        }
        String bookingId = "BKG" + bookingCounter++;
        Booking booking = new Booking(bookingId, flight, passenger);
        booking.seats.addAll(selectedSeats);
        bookings.put(bookingId, booking);
        System.out.println("Booking created successfully!");
        displayReceipt(booking);
        showAvailability(flight);
    }
    static void makePayment() {
        System.out.println("\n--- MAKE PAYMENT ---");
        String id = read("Booking ID: ").toUpperCase();
        Booking b = bookings.get(id);
        if (b == null || b.cancelled) {
            System.out.println("Valid active booking not found.");
            return;
        }
        if (b.payment != null && b.payment.paid) {
            System.out.println("Payment has already been completed.");
            return;
        }
        displayReceipt(b);
        System.out.println("1. Credit/Debit Card");
        System.out.println("2. UPI");
        System.out.println("3. Net Banking");
        int choice = readInt("Select Payment Method: ");
        String method;
        switch (choice) {
            case 1:
                method = "Card";
                break;
            case 2:
                method = "UPI";
                break;
            case 3:
                method = "Net Banking";
                break;
            default:
                System.out.println("Invalid payment method.");
                return;
        }
        b.payment = new Payment(b.getTotal(), method);
        System.out.println("Payment Successful!");
        System.out.printf("Amount Paid: Rs. %.2f%n", b.payment.amount);
        System.out.println("Method: " + b.payment.paymentMethod);
        System.out.println("Transaction ID: " + b.payment.transactionId);
    }
    static void checkIn() {
        System.out.println("\n--- CHECK-IN & BOARDING PASS ---");
        String id = read("Booking ID: ").toUpperCase();
        Booking b = bookings.get(id);
        if (b == null || b.cancelled) {
            System.out.println("Valid active booking not found.");
            return;
        }
        if (b.payment == null || !b.payment.paid) {
            System.out.println("Payment required before check-in!");
            return;
        }
        if (b.checkedIn) {
            System.out.println("Already checked in.");
            return;
        }
        b.checkedIn = true;
       System.out.println("Check-in successful!");
        new BoardingPass(b).display();
        showAvailability(b.flight);
    }
    static void cancelBooking() {
        System.out.println("\n--- CANCEL BOOKING ---");
        String id = read("Booking ID: ").toUpperCase();
        Booking b = bookings.get(id);
        if (b == null || b.cancelled) {
            System.out.println("Active booking not found.");
            return;
        }
        if (b.checkedIn) {
            System.out.println("Cannot cancel after check-in.");
            return; }
        for (Seat seat : b.seats) {
            seat.booked = false;
        }
        b.cancelled = true;
        System.out.println("Booking cancelled successfully!");
        System.out.println("Reserved seats have been released.");
        if (b.payment != null && b.payment.paid) {
            System.out.printf(
                "Refund due as per policy: Rs. %.2f%n",
                b.payment.amount);
            System.out.println("Note: Refund is not processed by this demo.");
        }
        showAvailability(b.flight);
    }
    static void showAvailability(Flight f) {
        System.out.println("Available Seats: "
            + f.aircraft.availableSeats() + "/"
            + f.aircraft.seats.size());
    }
    static void displayFlights() {
        System.out.println("\n---------------- FLIGHTS ----------------");
        if (flights.isEmpty()) {
            System.out.println("No flights available.");
            return;
        }
        System.out.printf("%-12s %-15s %-15s %-12s %-8s %-15s %-10s%n",
            "Flight No", "From", "To", "Date", "Time",
            "Seats(A/T)", "Fare");
        for (Flight f : flights.values()) {
System.out.printf("%-12s %-15s %-15s %-12s %-8s %-15s Rs.%-8.2f%n",f.flightNumber,
f.source,f.destination,f.date,f.time, f.aircraft.availableSeats() + "/" + f.aircraft.seats.size(),f.baseFare);
        }
        System.out.println("------------------------------------------");
    }
    public static void main(String[] args) {
        while (true) {
            System.out.println("\n====== AIRLINE RESERVATION SYSTEM ======");
            System.out.println("1. Add Flight");
            System.out.println("2. Add Passenger");
            System.out.println("3. Create Booking");
            System.out.println("4. Make Payment");
            System.out.println("5. Check-in & Issue Boarding Pass");
            System.out.println("6. Cancel Booking");
            System.out.println("7. Display Flights");
            System.out.println("8. Exit");
            int choice = readInt("Enter your choice: ");
            switch (choice) {
                case 1:
                    addFlight();
                    break;
                case 2:
                    addPassenger();
                    break;
                case 3:
                    createBooking();
                    break;
                case 4:
                    makePayment();
                    break;
                case 5:
                    checkIn();
                    break;
                case 6:
                    cancelBooking();
                    break;
                case 7:
                    displayFlights();
                    break;
                case 8:
            System.out.println("Thank you for using Airline Reservation System!");
                    sc.close();
                    return;
                default:
                    System.out.println("Invalid choice. Try again.");
            }
        }
    }
}

