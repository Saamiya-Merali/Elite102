import mysql.connector  # Importing the MySQL connector library

# Function to establish a connection to the MySQL database
def connect_db():
    return mysql.connector.connect(
        host='localhost',  # Hostname where the database is running
        user='root',  # Database username
        password='Jawaad123!',  # Database password
        database='project_schema'  # Name of the database
    )

# Function to create a new bank account in the database
def create_account(cursor):
    # Get user input for account information
    first_name = input("Enter your first name: ")
    last_name = input("Enter your last name: ")
    birth_date = input("Enter your birthdate (YYYY-MM-DD): ")
    password = input("Create a password: ")
    balance = float(input("Initial deposit amount: "))  # Initial deposit as a float value
    
    # Insert the new account into the database
    cursor.execute(
        "INSERT INTO bank_account (firstname, lastname, birthday, password, balance) VALUES (%s, %s, %s, %s, %s)",
        (first_name, last_name, birth_date, password, balance)  # Inserted data
    )
    print("Account created successfully!")  # Confirmation message

# Function to retrieve the account ID using the password
def get_account_id_from_password(cursor, password):
    cursor.execute("SELECT idbank_accounts FROM bank_account WHERE password = %s", (password,))
    result = cursor.fetchone()  # Fetch the first result
    return result[0] if result else None  # Return the account ID or None if not found

# Function to get the current balance of an account
def get_account_balance(cursor, account_id):
    cursor.execute("SELECT balance FROM bank_account WHERE idbank_accounts = %s", (account_id,))
    balance = cursor.fetchone()  # Fetch the balance
    return balance[0] if balance else None  # Return the balance or None

# Function to update the balance of a specific account
def update_account_balance(cursor, account_id, new_balance):
    cursor.execute("UPDATE bank_account SET balance = %s WHERE idbank_accounts = %s", 
                   (new_balance, account_id))  # Update the balance

# Function to handle various account operations
def account_operations(cursor, account_id):
    while True:
        # Prompt the user to choose an operation
        action = input("Choose an action (Deposit, Withdraw, Check Balance, Settings, Exit): ").lower()
        
        if action == 'check balance':
            balance = get_account_balance(cursor, account_id)  # Get current balance
            print(f"Your current balance is: {balance}")  # Display the balance
        
        elif action == 'deposit':
            deposit_amount = float(input("Deposit amount: "))  # Get deposit amount
            balance = get_account_balance(cursor, account_id)  # Get current balance
            new_balance = balance + deposit_amount  # Calculate new balance
            update_account_balance(cursor, account_id, new_balance)  # Update the balance
            print(f"Deposit successful! New balance: {new_balance}")  # Confirmation message
        
        elif action == 'withdraw':
            withdrawal_amount = float(input("Withdrawal amount: "))  # Get withdrawal amount
            balance = get_account_balance(cursor, account_id)  # Get current balance
            if withdrawal_amount > balance:  # Check if enough balance is available
                print("Insufficient balance!")  # Error message
            else:
                new_balance = balance - withdrawal_amount  # Calculate new balance
                update_account_balance(cursor, account_id, new_balance)  # Update the balance
                print(f"Withdrawal successful! New balance: {new_balance}")  # Confirmation message
        
        elif action == 'settings':
            change_account_settings(cursor, account_id)  # Open account settings
        
        elif action == 'exit':
            break  # Exit the loop
        
        else:
            print("Invalid action. Please try again.")  # Invalid action feedback

# Function to change account settings like name or delete the account
def change_account_settings(cursor, account_id):
    while True:
        # Prompt the user to choose a setting action
        setting_action = input("Choose a setting action (Change Name, Delete Account, Exit): ").lower()
        
        if setting_action == 'change name':
            first_name = input("New first name: ")  # Get new first name
            last_name = input("New last name: ")  # Get new last name
            cursor.execute("UPDATE bank_account SET firstname = %s, lastname = %s WHERE idbank_accounts = %s", 
                           (first_name, last_name, account_id))  # Update the name
            print("Name updated successfully.")  # Confirmation message
        
        elif setting_action == 'delete account':
            cursor.execute("DELETE FROM bank_account WHERE idbank_accounts = %s", (account_id,))  # Delete the account
            print("Account deleted successfully.")  # Confirmation message
            break  # Exit the loop
        
        elif setting_action == 'exit':
            break  # Exit the loop
        
        else:
            print("Invalid setting action. Please try again.")  # Invalid action feedback

# Function for user login to validate their credentials
def login(cursor):
    while True:
        password = input("Enter your password: ")  # Prompt user for password
        account_id = get_account_id_from_password(cursor, password)  # Retrieve account ID
        
        if account_id:  # If the account ID is valid
            print("Login successful.")  # Login confirmation
            account_operations(cursor, account_id)  # Start account operations
            break  # Exit the loop
        
        else:
            print("Incorrect password. Please try again.")  # Incorrect password feedback

# Main function to start the program
def main():
    db = connect_db()  # Establish database connection
    cursor = db.cursor(buffered=True)  # Create a cursor with buffering
    db.autocommit = True  # Enable autocommit
    
    while True:
        # Welcome message
        print("Welcome to the Bank!")
        
        # Ask if the user has an account
        has_account = input("Do you have an account? (Y/N): ").upper()  # Convert to uppercase
        
        if has_account == 'Y':  # If the user has an account
            login(cursor)  # Proceed to login
        
        else:  # If the user does not have an account
            create_new_account = input("Would you like to create a new account? (Y/N): ").upper()  # Ask to create a new account
            if create_new_account == 'Y':
                create_account(cursor)  # Create a new account
        
        # Ask if the user wants to exit
        if input("Would you like to exit? (Y/N): ").upper() == 'Y':  # If the user wants to exit
            break  # Break the loop
    
    print("Thank you for visiting the Bank! Come back soon.")  # Exit message

# Entry point for the program
if __name__ == "__main__":
    main()  # Execute the main function
