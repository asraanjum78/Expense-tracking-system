#include <iostream>
#include <string>

using namespace std;

// Structure for expense items (inner linked list)
struct ExpenseNode {
    string description;
    double amount;
    string date;
    ExpenseNode* next;

    // Constructor
    ExpenseNode(const string& desc, double amt, const string& dt) {
        description = desc;
        amount = amt;
        date = dt;
        next = NULL;
    }
};

// Structure for months (outer linked list)
struct MonthNode {
    string month;
    int year;
    ExpenseNode* expenseHead; // Points to the head of expense linked list
    MonthNode* next;

    // Constructor
    MonthNode(const string& m, int y) {
        month = m;
        year = y;
        expenseHead = NULL;
        next = NULL;
    }
};

class ExpenseTracker {
private:
    MonthNode* head;

public:
    // Constructor
    ExpenseTracker() {
        head = NULL;
    }

    // Destructor
    ~ExpenseTracker() {
        clearAll();
    }

    // Create a new month or get existing one
    MonthNode* findOrCreateMonth(const string& month, int year) {
        // Check if month already exists
        MonthNode* current = head;
        while (current) {
            if (current->month == month && current->year == year) {
                return current;
            }
            current = current->next;
        }

        // If month doesn't exist, create a new one
        MonthNode* newMonth = new MonthNode(month, year);
        
        // If list is empty, set as head
        if (!head) {
            head = newMonth;
            return head;
        }

        // Otherwise, add to the end of the list
        current = head;
        while (current->next) {
            current = current->next;
        }
        current->next = newMonth;
        return newMonth;
    }

    // CREATE: Add a new expense to a specific month
    void addExpense(const string& month, int year, const string& description, double amount, const string& date) {
        MonthNode* monthNode = findOrCreateMonth(month, year);
        ExpenseNode* newExpense = new ExpenseNode(description, amount, date);

        // If no expenses for this month yet
        if (!monthNode->expenseHead) {
            monthNode->expenseHead = newExpense;
            return;
        }

        // Otherwise, add to the end of the expense list
        ExpenseNode* current = monthNode->expenseHead;
        while (current->next) {
            current = current->next;
        }
        current->next = newExpense;
    }

    // READ: Display all expenses for a specific month
    void displayMonthExpenses(const string& month, int year) {
        MonthNode* monthNode = findMonth(month, year);
        if (!monthNode) {
            cout << "Month not found!" << endl;
            return;
        }

        cout << "\n=== Expenses for " << month << " " << year << " ===\n";
        
        ExpenseNode* current = monthNode->expenseHead;
        if (!current) {
            cout << "No expenses recorded for this month.\n";
            return;
        }

        int count = 1;
        double total = 0;
        
        cout << "No. Description                Amount ($)      Date\n";
        cout << "----------------------------------------------------\n";
        
        while (current) {
            cout << count++ << ".  ";
            cout << current->description;
            // Add spaces to align columns (simple formatting without iomanip)
            for (int i = 0; i < 25 - current->description.length(); i++) {
                cout << " ";
            }
            cout << current->amount;
            // Calculate amount string length manually
            int amountLen = 1; // At least 1 digit
            int tempAmount = (int)current->amount;
            while (tempAmount >= 10) {
                tempAmount /= 10;
                amountLen++;
            }
            for (int i = 0; i < 15 - amountLen; i++) {
                cout << " ";
            }
            cout << current->date << endl;
            
            total += current->amount;
            current = current->next;
        }
        
        cout << "----------------------------------------------------\n";
        cout << "Total expenses: $" << total << endl;
    }

    // Display most expensive item for a month
    void displayMostExpensiveItem(const string& month, int year) {
        MonthNode* monthNode = findMonth(month, year);
        if (!monthNode || !monthNode->expenseHead) {
            cout << "No expenses found for " << month << " " << year << "!" << endl;
            return;
        }

        ExpenseNode* current = monthNode->expenseHead;
        ExpenseNode* mostExpensive = current;

        while (current) {
            if (current->amount > mostExpensive->amount) {
                mostExpensive = current;
            }
            current = current->next;
        }

        cout << "\n=== Most Expensive Item for " << month << " " << year << " ===\n";
        cout << "Description: " << mostExpensive->description << endl;
        cout << "Amount: $" << mostExpensive->amount << endl;
        cout << "Date: " << mostExpensive->date << endl;
    }

    // UPDATE: Update an expense
    bool updateExpense(const string& month, int year, const string& oldDesc, 
                      const string& newDesc, double newAmount, const string& newDate) {
        MonthNode* monthNode = findMonth(month, year);
        if (!monthNode) {
            return false;
        }

        ExpenseNode* current = monthNode->expenseHead;
        while (current) {
            if (current->description == oldDesc) {
                current->description = newDesc;
                current->amount = newAmount;
                current->date = newDate;
                return true;
            }
            current = current->next;
        }

        return false; // Expense not found
    }

    // DELETE: Delete an expense
    bool deleteExpense(const string& month, int year, const string& description) {
        MonthNode* monthNode = findMonth(month, year);
        if (!monthNode) {
            return false;
        }

        // If the expense to delete is the head
        if (monthNode->expenseHead && monthNode->expenseHead->description == description) {
            ExpenseNode* temp = monthNode->expenseHead;
            monthNode->expenseHead = monthNode->expenseHead->next;
            delete temp;
            return true;
        }

        // Search for the expense in the list
        ExpenseNode* current = monthNode->expenseHead;
        while (current && current->next) {
            if (current->next->description == description) {
                ExpenseNode* temp = current->next;
                current->next = current->next->next;
                delete temp;
                return true;
            }
            current = current->next;
        }

        return false; // Expense not found
    }

    // Delete a month
    bool deleteMonth(const string& month, int year) {
        // If the month to delete is the head
        if (head && head->month == month && head->year == year) {
            MonthNode* temp = head;
            head = head->next;
            
            // Delete all expenses for this month
            clearExpenses(temp->expenseHead);
            delete temp;
            return true;
        }

        // Search for the month in the list
        MonthNode* current = head;
        while (current && current->next) {
            if (current->next->month == month && current->next->year == year) {
                MonthNode* temp = current->next;
                current->next = current->next->next;
                
                // Delete all expenses for this month
                clearExpenses(temp->expenseHead);
                delete temp;
                return true;
            }
            current = current->next;
        }

        return false; // Month not found
    }

    // Helper function to find a month
    MonthNode* findMonth(const string& month, int year) {
        MonthNode* current = head;
        while (current) {
            if (current->month == month && current->year == year) {
                return current;
            }
            current = current->next;
        }
        return NULL; // Month not found
    }

    // Display all months
    void displayAllMonths() {
        if (!head) {
            cout << "\nNo months recorded yet.\n";
            return;
        }

        cout << "\n=== All Recorded Months ===\n";
        MonthNode* current = head;
        int count = 1;
        
        while (current) {
            cout << count++ << ". " << current->month << " " << current->year << endl;
            current = current->next;
        }
    }

    // Display summary of all months
    void displaySummary() {
        if (!head) {
            cout << "\nNo expenses recorded yet.\n";
            return;
        }

        cout << "\n=== Expense Summary ===\n";
        cout << "Month/Year           Total ($)   Most Expensive Item             Amount ($)\n";
        cout << "------------------------------------------------------------------------\n";
        
        MonthNode* currentMonth = head;
        
        while (currentMonth) {
            double total = 0;
            ExpenseNode* mostExpensive = NULL;
            ExpenseNode* currentExpense = currentMonth->expenseHead;
            
            while (currentExpense) {
                total += currentExpense->amount;
                if (!mostExpensive || currentExpense->amount > mostExpensive->amount) {
                    mostExpensive = currentExpense;
                }
                currentExpense = currentExpense->next;
            }
            
            // Create month/year string without using to_string
            string monthYear = currentMonth->month + " ";
            
            // Convert year to string manually
            int yearValue = currentMonth->year;
            string yearStr = "";
            
            // Handle negative years (unlikely but being thorough)
            if (yearValue < 0) {
                yearStr = "-";
                yearValue = -yearValue;
            }
            
            // Convert year to string by extracting digits
            int digits[10]; // Max 10 digits should be enough for a year
            int digitCount = 0;
            
            // Special case for zero
            if (yearValue == 0) {
                yearStr += "0";
            } else {
                // Extract digits
                while (yearValue > 0) {
                    digits[digitCount++] = yearValue % 10;
                    yearValue /= 10;
                }
                
                // Add digits in reverse order
                for (int i = digitCount - 1; i >= 0; i--) {
                    yearStr += (char)(digits[i] + '0');
                }
            }
            
            monthYear += yearStr;
            cout << monthYear;
            for (int i = 0; i < 20 - monthYear.length(); i++) {
                cout << " ";
            }
            cout << total;
            // Calculate total string length manually
            int totalLen = 1; // At least 1 digit
            int tempTotal = (int)total;
            while (tempTotal >= 10) {
                tempTotal /= 10;
                totalLen++;
            }
            for (int i = 0; i < 12 - totalLen; i++) {
                cout << " ";
            }
                 
            if (mostExpensive) {
                cout << mostExpensive->description;
                for (int i = 0; i < 30 - mostExpensive->description.length(); i++) {
                    cout << " ";
                }
                cout << mostExpensive->amount;
            } else {
                cout << "N/A                           0.00";
            }
            cout << endl;
            
            currentMonth = currentMonth->next;
        }
    }

    // Helper function to clear expenses for a month
    void clearExpenses(ExpenseNode* expenseHead) {
        ExpenseNode* current = expenseHead;
        while (current) {
            ExpenseNode* temp = current;
            current = current->next;
            delete temp;
        }
    }

    // Clear all data
    void clearAll() {
        MonthNode* current = head;
        while (current) {
            // Clear all expenses for this month
            clearExpenses(current->expenseHead);
            
            // Move to next month and delete current
            MonthNode* temp = current;
            current = current->next;
            delete temp;
        }
        head = NULL;
    }
};

// Function to validate double input
double getValidAmount() {
    double amount;
    while (!(cin >> amount) || amount < 0) {
        cout << "Invalid input. Please enter a positive number: ";
        cin.clear();
        cin.ignore(10000, '\n');
    }
    cin.ignore(); // Clear the newline character
    return amount;
}

// Function to validate year input
int getValidYear() {
    int year;
    while (!(cin >> year) || year < 1900 || year > 2100) {
        cout << "Invalid input. Please enter a year between 1900 and 2100: ";
        cin.clear();
        cin.ignore(10000, '\n');
    }
    cin.ignore(); // Clear the newline character
    return year;
}

// Main menu function
void displayMenu() {
    cout << "\n=== Expense Tracker Menu ===\n";
    cout << "1. Add a new expense\n";
    cout << "2. View expenses for a month\n";
    cout << "3. View most expensive item for a month\n";
    cout << "4. Update an expense\n";
    cout << "5. Delete an expense\n";
    cout << "6. Delete a month\n";
    cout << "7. View all months\n";
    cout << "8. View expense summary\n";
    cout << "9. Exit\n";
    cout << "Enter your choice: ";
}

int main() {
    ExpenseTracker tracker;
    int choice;
    string month, description, date, oldDescription;
    int year;
    double amount;

    cout << "====== Expense Tracking System ======\n";
    
    // Add some sample data
    tracker.addExpense("January", 2025, "Rent", 1200.00, "01/01/2025");
    tracker.addExpense("January", 2025, "Groceries", 350.75, "05/01/2025");
    tracker.addExpense("January", 2025, "Utilities", 180.50, "10/01/2025");
    tracker.addExpense("February", 2025, "Rent", 1200.00, "01/02/2025");
    tracker.addExpense("February", 2025, "Shopping", 525.99, "15/02/2025");
    
    do {
        displayMenu();
        while (!(cin >> choice) || choice < 1 || choice > 9) {
            cout << "Invalid choice. Please enter a number between 1 and 9: ";
            cin.clear();
            cin.ignore(10000, '\n');
        }
        cin.ignore(); // Clear the newline character

        switch (choice) {
            case 1: // Add a new expense
                cout << "\nEnter month (e.g., January): ";
                getline(cin, month);
                
                cout << "Enter year: ";
                year = getValidYear();
                
                cout << "Enter expense description: ";
                getline(cin, description);
                
                cout << "Enter amount: $";
                amount = getValidAmount();
                
                cout << "Enter date (e.g., DD/MM/YYYY): ";
                getline(cin, date);
                
                tracker.addExpense(month, year, description, amount, date);
                cout << "Expense added successfully!\n";
                break;
                
            case 2: // View expenses for a month
                cout << "\nEnter month: ";
                getline(cin, month);
                
                cout << "Enter year: ";
                year = getValidYear();
                
                tracker.displayMonthExpenses(month, year);
                break;
                
            case 3: // View most expensive item
                cout << "\nEnter month: ";
                getline(cin, month);
                
                cout << "Enter year: ";
                year = getValidYear();
                
                tracker.displayMostExpensiveItem(month, year);
                break;
                
            case 4: // Update an expense
                cout << "\nEnter month: ";
                getline(cin, month);
                
                cout << "Enter year: ";
                year = getValidYear();
                
                tracker.displayMonthExpenses(month, year);
                
                cout << "\nEnter the description of the expense to update: ";
                getline(cin, oldDescription);
                
                cout << "Enter new description: ";
                getline(cin, description);
                
                cout << "Enter new amount: $";
                amount = getValidAmount();
                
                cout << "Enter new date (e.g., DD/MM/YYYY): ";
                getline(cin, date);
                
                if (tracker.updateExpense(month, year, oldDescription, description, amount, date)) {
                    cout << "Expense updated successfully!\n";
                } else {
                    cout << "Expense not found!\n";
                }
                break;
                
            case 5: // Delete an expense
                cout << "\nEnter month: ";
                getline(cin, month);
                
                cout << "Enter year: ";
                year = getValidYear();
                
                tracker.displayMonthExpenses(month, year);
                
                cout << "\nEnter the description of the expense to delete: ";
                getline(cin, description);
                
                if (tracker.deleteExpense(month, year, description)) {
                    cout << "Expense deleted successfully!\n";
                } else {
                    cout << "Expense not found!\n";
                }
                break;
                
            case 6: // Delete a month
                cout << "\nEnter month: ";
                getline(cin, month);
                
                cout << "Enter year: ";
                year = getValidYear();
                
                if (tracker.deleteMonth(month, year)) {
                    cout << "Month deleted successfully!\n";
                } else {
                    cout << "Month not found!\n";
                }
                break;
                
            case 7: // View all months
                tracker.displayAllMonths();
                break;
                
            case 8: // View expense summary
                tracker.displaySummary();
                break;
                
            case 9: // Exit
                cout << "\nThank you for using the Expense Tracker. Goodbye!\n";
                break;
        }
    } while (choice != 9);

    return 0;
}
