import java.util.*;

class Vehicle {
    String number;
    String type; // Car or Bike
    long entryTime;
    long exitTime;
    double charge;

    Vehicle(String number, String type) {
        this.number = number;
        this.type = type;
        this.entryTime = System.currentTimeMillis();
    }
}

public class ParkingLotmanagement{
    static Scanner sc = new Scanner(System.in);
    static int totalCarSlots = 10;
    static int totalBikeSlots = 10;
    static Map<String, Vehicle> parkedVehicles = new HashMap<>();

    public static void main(String[] args) {
        while (true) {
           
            System.out.println("\n=== NARMADA ENTERPRISES PARKING MANAGEMENT ===");
            System.out.println("\t=== OWNER - LAKSHYA PUROHIT ===");
            System.out.println("\n1. Vehicle Entry");
            System.out.println("2. Vehicle Exit");
            System.out.println("3. Show Available Slots");
            System.out.println("4. Show Parked Vehicles");
            System.out.println("5. Exit");
            System.out.print("Choose option: ");
            int choice = sc.nextInt();

            switch (choice) {
                case 1: vehicleEntry();
                break;
                case 2: vehicleExit();
                break;
                case 3: showAvailableSlots();
                break;
                case 4: showParkedVehicles();
                break;
                case 5: System.out.println("Goodbye Buddy!");
                return;
                default: System.out.println("Invalid choice.");
            }
        }
    }

    static void vehicleEntry() {
        sc.nextLine(); // clear buffer
        System.out.print("Enter Vehicle Number: ");
        String number = sc.nextLine();
        System.out.print("Enter Vehicle Type (Car/Bike): ");
        String type = sc.nextLine().trim();

        if (type.equalsIgnoreCase("Car") && countType("Car") >= totalCarSlots) {
            System.out.println(" No car slots available!");
            return;
        }
        if (type.equalsIgnoreCase("Bike") && countType("Bike") >= totalBikeSlots) {
            System.out.println(" No bike slots available!");
            return;
        }

        Vehicle v = new Vehicle(number, type);
        parkedVehicles.put(number, v);
        System.out.println(" Vehicle parked successfully.");
    }

    static void vehicleExit() {
        sc.nextLine(); // clear buffer
        System.out.print("Enter Vehicle Number: ");
        String number = sc.nextLine();
        Vehicle v = parkedVehicles.get(number);

        if (v == null) {
            System.out.println(" Vehicle not found.");
            return;
        }

        v.exitTime = System.currentTimeMillis();
        long durationMs = v.exitTime - v.entryTime;
        long minutes = durationMs / (1000 * 60);

        if (minutes == 0) minutes = 1; // minimum 1 minute charge

        // Charge: Car = ₹50/hour, Bike = ₹20/hour
        double ratePerHour = v.type.equalsIgnoreCase("Car") ? 50 : 20;
        v.charge = ratePerHour * (minutes / 60.0);

        System.out.printf("Vehicle %s is exited. \nTime parked: %d min and Charge = Rupees %.2f\n",
                v.number, minutes, v.charge);

        parkedVehicles.remove(number);
    }

    static void showAvailableSlots() {
        int carUsed = countType("Car");
        int bikeUsed = countType("Bike");
        System.out.println(" Car slots available: " + (totalCarSlots - carUsed));
        System.out.println(" Bike slots available: " + (totalBikeSlots - bikeUsed));
    }

    static void showParkedVehicles() {
        if (parkedVehicles.isEmpty()) {
            System.out.println("No vehicles currently parked.");
            return;
        }
        System.out.println("Currently Parked Vehicles:");
        for (Vehicle v : parkedVehicles.values()) {
            System.out.println(" - " + v.type + " | Number: " + v.number);
        }
    }

    static int countType(String type) {
        int count = 0;
        for (Vehicle v : parkedVehicles.values()) {
            if (v.type.equalsIgnoreCase(type)) count++;
        }
        return count;
    }
}
