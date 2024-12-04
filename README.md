# DS-Project
#include <iostream>
#include <string>
#include <vector>
#include <stack>
#include <queue>
#include <algorithm>
#include <iomanip>
#include <limits>

using namespace std;

// Utility Function: Clear Screen
void clearScreen() {
    system("CLS"); // Clear screen for Windows
}

// Room Types Enum
enum RoomType {
    SINGLE_BED,
    DOUBLE_BED,
    DELUXE_ROOM
};

// Utility Functions
string getRoomTypeString(RoomType type) {
    switch (type) {
        case SINGLE_BED: return "Single Bed";
        case DOUBLE_BED: return "Double Bed";
        case DELUXE_ROOM: return "Deluxe Room";
        default: return "Unknown";
    }
}

// Linked List Node for Room
struct RoomNode {
    int roomNumber;
    RoomType type;
    double pricePerNight;
    bool isAvailable;
    bool isUnderMaintenance;
    RoomNode* next;

    RoomNode(int number, RoomType t, double price)
        : roomNumber(number), type(t), pricePerNight(price),
          isAvailable(true), isUnderMaintenance(false), next(nullptr) {}

    void displayDetails() const {
        cout << "Room " << roomNumber
             << " - " << getRoomTypeString(type)
             << " - Rs. " << pricePerNight << " per night - "
             << (isAvailable ? "Available" : "Not Available")
             << (isUnderMaintenance ? " (Under Maintenance)" : "") << endl;
    }
};

// Binary Search Tree Node for Room Organization
struct TreeNode {
    RoomNode* room;
    TreeNode* left;
    TreeNode* right;

    TreeNode(RoomNode* r) : room(r), left(nullptr), right(nullptr) {}
};

class RoomTree {
private:
    TreeNode* root;

    void inorderTraversal(TreeNode* node) const {
        if (!node) return;
        inorderTraversal(node->left);
        node->room->displayDetails();
        inorderTraversal(node->right);
    }

    TreeNode* insert(TreeNode* node, RoomNode* room) {
        if (!node) return new TreeNode(room);
        if (room->roomNumber < node->room->roomNumber)
            node->left = insert(node->left, room);
        else
            node->right = insert(node->right, room);
        return node;
    }

    RoomNode* search(TreeNode* node, int roomNumber) const {
        if (!node) return nullptr;
        if (roomNumber == node->room->roomNumber)
            return node->room;
        if (roomNumber < node->room->roomNumber)
            return search(node->left, roomNumber);
        return search(node->right, roomNumber);
    }

public:
    RoomTree() : root(nullptr) {}

    void addRoom(RoomNode* room) {
        root = insert(root, room);
    }

    RoomNode* findRoom(int roomNumber) const {
        return search(root, roomNumber);
    }

    void displayAllRooms() const {
        inorderTraversal(root);
    }
};

// Meal Class
class Meal {
public:
    string name;
    double price;
    string description;

    Meal(string n, double p, string desc = "") : name(n), price(p), description(desc) {}

    void displayDetails() const {
        cout << name << " - Rs. " << price;
        if (!description.empty()) {
            cout << " (" << description << ")";
        }
        cout << endl;
    }
};

// Customer Class
class Customer {
public:
    string name;
    string phoneNumber;
    string email;
    int roomNumber;

    Customer(string n, string phone, string em, int r)
        : name(n), phoneNumber(phone), email(em), roomNumber(r) {}

    void displayDetails() const {
        cout << "Name: " << name << ", Phone: " << phoneNumber
             << ", Email: " << email << ", Room: " << roomNumber << endl;
    }
};

// Reservation Class
class Reservation {
public:
    Customer customer;
    RoomNode* room;
    vector<Meal> orderedMeals;
    double totalBill;

    Reservation(Customer c, RoomNode* r) : customer(c), room(r), totalBill(0) {}

    // Method to add a meal to the reservation
    void addMeal(const Meal& meal) {
        orderedMeals.push_back(meal);
        totalBill += meal.price;
    }

    // Method to calculate the total bill (room price + meals)
    void calculateTotalBill(int days) {
        totalBill += room->pricePerNight * days; // Room cost
    }

    // Method to display the reservation details
    void displayReservation() const {
        customer.displayDetails();
        room->displayDetails();
        cout << "Meals Ordered: " << orderedMeals.size() << endl;
        
        if (orderedMeals.size() > 0) {
            cout << "Meal Details:\n";
            for (const auto& meal : orderedMeals) {
                cout << " - " << meal.name << " (Rs. " << meal.price << ")\n";
            }
        }

        cout << "Total Bill: Rs. " << totalBill << endl;
    }
};
// Hotel Management System
class HotelManagementSystem {
private:
    RoomNode* roomHead;
    RoomTree roomTree;
    vector<Meal> meals;
    vector<Customer> customers;
    vector<Reservation> reservations;
    queue<Customer> waitlist;
    stack<string> adminActions;
    

public:
    HotelManagementSystem() : roomHead(nullptr) {
     

        // Initialize Rooms
        addRoom(101, SINGLE_BED, 2500);
        addRoom(102, SINGLE_BED, 2500);
        addRoom(201, DOUBLE_BED, 4000);
        addRoom(202, DOUBLE_BED, 4000);
        addRoom(301, DELUXE_ROOM, 6000);
        addRoom(302, DELUXE_ROOM, 6000);
    }

    ~HotelManagementSystem() {
        RoomNode* current = roomHead;
        while (current) {
            RoomNode* toDelete = current;
            current = current->next;
            delete toDelete;
        }
    }

    void addRoom(int number, RoomType type, double price) {
        RoomNode* newRoom = new RoomNode(number, type, price);
        newRoom->next = roomHead;
        roomHead = newRoom;
        roomTree.addRoom(newRoom);
        adminActions.push("Added Room " + to_string(number));
    }

    void viewAvailableRooms() const {
        cout << "\nAvailable Rooms:\n";
        roomTree.displayAllRooms();
        cin.ignore();
        cin.get();
    }

    void reserveRoom() {
        viewAvailableRooms();
        int roomNumber, days;
        string name, phone, email;

        cout << "Enter Room Number to Reserve: ";
        cin >> roomNumber;

        RoomNode* room = roomTree.findRoom(roomNumber);
        if (!room || !room->isAvailable || room->isUnderMaintenance) {
            cout << "Room not available! Adding customer to waitlist.\n";
            cout << "Enter Customer Details\n";
            cin.ignore();
            cout << "Name: ";
            getline(cin, name);
            cout << "Phone: ";
            getline(cin, phone);
            cout << "Email: ";
            getline(cin, email);
            Customer customer(name, phone, email, roomNumber);
            waitlist.push(customer);
            return;
        }

        cout << "Enter Customer Details\n";
        cin.ignore();
        cout << "Name: ";
        getline(cin, name);
        cout << "Phone: ";
        getline(cin, phone);
        cout << "Email: ";
        getline(cin, email);
        cout << "Number of Days: ";
        cin >> days;

        Customer customer(name, phone, email, roomNumber);
        customers.push_back(customer);
        Reservation reservation(customer, room);

        reservation.calculateTotalBill(days);
        reservations.push_back(reservation);
        room->isAvailable = false;

        cout << "Reservation successful! Total Bill: Rs. " << reservation.totalBill << endl;
        cin.ignore();
        cin.get();
    }

    void cancelReservation() {
    int roomNumber;
    cout << "Enter Room Number for Check-Out: ";
    cin >> roomNumber;

    auto it = find_if(reservations.begin(), reservations.end(),
                      [roomNumber](const Reservation& res) {
                          return res.room->roomNumber == roomNumber;
                      });

    if (it != reservations.end()) {
        Reservation& reservation = *it;
        cout << "\nGenerating Bill for Customer:\n";
        reservation.customer.displayDetails();
        reservation.room->displayDetails();

        cout << "\nOrdered Meals:\n";
        for (const auto& meal : reservation.orderedMeals) {
            meal.displayDetails();
        }

        cout << "\nTotal Bill: Rs. " << reservation.totalBill << endl;
        // Mark the room as available
        reservation.room->isAvailable = true;
        // Handle waitlist if applicable
        if (!waitlist.empty()) {
            Customer customer = waitlist.front();
            waitlist.pop();
            Reservation newReservation(customer, reservation.room);
            reservations.push_back(newReservation);
            reservation.room->isAvailable = false;
            cout << "\nCustomer " << customer.name
                 << " from waitlist is now reserved in Room " << roomNumber << ".\n";
        }
        // Remove the reservation
        reservations.erase(it);
        cout << "Check-Out Complete!\n";
    } else {
        cout << "No reservation found for Room " << roomNumber << ".\n";
    }

    cin.ignore();
    cin.get();
}


    void viewReservations() const {
        cout << "\nCurrent Reservations:\n";
        for (const auto& res : reservations) {
            res.displayReservation();
            cout << "----------------------\n";
        }
        cin.ignore();
        cin.get();
    }

    void addMaintenanceRequest() {
        int roomNumber;
        cout << "Enter Room Number to Mark for Maintenance: ";
        cin >> roomNumber;

        RoomNode* room = roomTree.findRoom(roomNumber);
        if (room) {
            room->isUnderMaintenance = true;
            cout << "Room " << roomNumber << " is now under maintenance.\n";
            adminActions.push("Room " + to_string(roomNumber) + " marked for maintenance.");
        } else {
            cout << "Room not found.\n";
        }
        cin.ignore();
        cin.get();
    }

    void removeMaintenanceRequest() {
        int roomNumber;
        cout << "Enter Room Number to Remove from Maintenance: ";
        cin >> roomNumber;

        RoomNode* room = roomTree.findRoom(roomNumber);
        if (room) {
            room->isUnderMaintenance = false;
            cout << "Room " << roomNumber << " is now available.\n";
            adminActions.push("Room " + to_string(roomNumber) + " removed from maintenance.");
        } else {
            cout << "Room not found.\n";
        }
        cin.ignore();
        cin.get();
    }
    void viewWaitlist() const {
        cout << "\nWaitlist:\n";
        if (waitlist.empty()) {
            cout << "No customers in the waitlist.\n";
        } else {
            queue<Customer> tempQueue = waitlist; // Copy the queue to preserve the original order
            while (!tempQueue.empty()) {
                Customer customer = tempQueue.front();
                customer.displayDetails();
                tempQueue.pop();
                cout << "----------------------\n";
            }
        }
        cin.ignore();
        cin.get();
    }    

    void viewAdminActions() const {
        cout << "\nRecent Admin Actions:\n";
        stack<string> tempActions = adminActions;
        while (!tempActions.empty()) {
            cout << tempActions.top() << endl;
            tempActions.pop();
        }
        cin.ignore();
        cin.get();
    }
    void viewMeals() {
    clearScreen();
    cout << "Eating Items and Snacks\n\n";

    cout << "Breakfast Options:\n";
    cout << "1. Pancakes with Maple Syrup\n";
    cout << "2. Scrambled Eggs with Toast\n";
    cout << "3. Fresh Fruit Salad\n";
    cout << "4. Masala Dosa with Sambar\n";
    cout << "5. Paratha with Yogurt\n\n";

    cout << "Lunch Options:\n";
    cout << "1. Grilled Chicken with Rice\n";
    cout << "2. Vegetable Biryani with Raita\n";
    cout << "3. Butter Chicken with Naan\n";
    cout << "4. Caesar Salad\n";
    cout << "5. Fish Curry with Steamed Rice\n\n";

    cout << "Dinner Options:\n";
    cout << "1. Spaghetti Bolognese\n";
    cout << "2. Mutton Korma with Paratha\n";
    cout << "3. Paneer Butter Masala with Roti\n";
    cout << "4. Chicken Steak with Mashed Potatoes\n";
    cout << "5. Thai Green Curry with Jasmine Rice\n\n";

    cout << "Press Enter to return to the main menu.\n";
    cin.ignore();
    cin.get();
}
    void adminMenu() {
        int choice;
        do {
            clearScreen();
            cout << "Admin Menu:\n";
            cout << "1. View All Rooms\n";
            cout << "2. Add New Room\n";
            cout << "3. Add Maintenance Request\n";
            cout << "4. Remove Maintenance Request\n";
            cout << "5. View Admin Actions\n";
            cout << "6. Exit\n";
            cout << "Enter choice: ";
            cin >> choice;

            switch (choice) {
                case 1: viewAvailableRooms(); break;
                case 2: {
                    int number;
                    double price;
                    int type;
                    cout << "Enter Room Number: ";
                    cin >> number;
                    cout << "Enter Room Price per Night: ";
                    cin >> price;
                    cout << "Enter Room Type (0: Single, 1: Double, 2: Deluxe): ";
                    cin >> type;
                    addRoom(number, static_cast<RoomType>(type), price);
                    break;
                }
                case 3: addMaintenanceRequest(); break;
                case 4: removeMaintenanceRequest(); break;
                case 5: viewAdminActions(); break;
            }
        } while (choice != 6);
    }
    

 void start() {
    int choice;
    do {
        clearScreen();
        cout << "Hotel Management System\n";
        cout << "1. Admin Menu\n";
        cout << "2. Reserve Room\n";
        cout << "3. Check-Out & Create Bill\n";
        cout << "4. View Reservations\n";
        cout << "5. View Waitlist\n";
        cout << "6. View Meals\n"; // New option for viewing meals
        cout << "7. Exit\n";
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice) {
            case 1: adminMenu(); break;
            case 2: reserveRoom(); break;
            case 3: cancelReservation(); break;
            case 4: viewReservations(); break;
            case 5: viewWaitlist(); break;
            case 6: viewMeals(); break; // Call the viewMeals function
        }
    } while (choice != 7);
}

    void viewMeals() const {
    clearScreen();
    cout << "\nAvailable Meals:\n";
    if (meals.empty()) {
        cout << "No meals available currently.\n";
    } else {
        for (const auto& meal : meals) {
            cout << meal.name << " - Rs. " << fixed << setprecision(2) << meal.price;
            if (!meal.description.empty()) {
                cout << " (" << meal.description << ")";
            }
            cout << endl;
        }
    }
    cout << "\nPress Enter to return to the menu.";
    cin.ignore();
    cin.get();
}


};

// Main Function
int main() {
    HotelManagementSystem system;
    system.start();
    return 0;
}
