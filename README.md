from datetime import datetime

class Car:
    def __init__(self, make, model, plate_number, available=True):
        self.make = make
        self.model = model
        self.plate_number = plate_number
        self.available = available
        self.rental_time = None

class RentalRecord:
    def __init__(self, user, car, plate_number, rental_time, return_time=None):
        self.user = user
        self.car = car
        self.plate_number = plate_number
        self.rental_time = rental_time
        self.return_time = return_time

class User:
    def __init__(self, username, password):
        self.username = username
        self.password = password

class CarRentalSystem:
    def __init__(self, currency='PHP'):
        self.cars = []
        self.users = {}
        self.rental_records = []
        self.currency = currency
        self.current_user = None

    def add_car(self, car):
        self.cars.append(car)

    def display_available_cars(self):
        available_cars = [car for car in self.cars if car.available]
        if available_cars:
            for car in available_cars:
                print(f"{car.make} {car.model} ({car.plate_number}) is available.")
        else:
            print("No cars available.")

    def rent_car(self, plate_number, rental_hours):
        for car in self.cars:
            if car.plate_number == plate_number and car.available:
                car.available = False
                car.rental_time = datetime.now()
                rental_record = RentalRecord(self.current_user, car, plate_number, car.rental_time)
                self.rental_records.append(rental_record)
                rental_price = self.calculate_rental_price(rental_hours)
                print(f"{self.current_user.username}, you have rented a {car.make} {car.model} ({car.plate_number}). "
                      f"Rental duration: {rental_hours} hours. Rental Price: ₱{rental_price}.")
                return
        print("Car not available for rent.")

    def calculate_rental_price(self, rental_hours):
        prices = {6: 500, 12: 1000, 24: 2000}
        return prices.get(rental_hours, 0)

    def return_car(self, plate_number):
        for record in self.rental_records:
            if record.plate_number == plate_number and record.return_time is None:
                record.return_time = datetime.now()
                record.car.available = True
                rental_duration = record.return_time - record.rental_time
                rental_hours = int(rental_duration.total_seconds() / 3600)
                rental_price = self.calculate_rental_price(rental_hours)
                print(f"{record.user.username}, you have returned the {record.car.make} {record.car.model} "
                      f"({record.plate_number}). Rental duration: {rental_hours} hours. Rental Price: ₱{rental_price}.")
                return
        print("Car not found in rental records or already returned.")

    def register_user(self, username, password):
        if username not in self.users:
            new_user = User(username, password)
            self.users[username] = new_user
            print(f"User {username} registered successfully.")
        else:
            print("Username already exists. Please choose a different username.")

    def login(self, username, password):
        if username in self.users and self.users[username].password == password:
            print(f"Welcome back, {username}!")
            self.current_user = self.users[username]
        else:
            print("Invalid username or password.")

    def display_rental_history(self):
        if self.current_user:
            print(f"Rental history for {self.current_user.username}:")
            for record in self.rental_records:
                if record.user == self.current_user:
                    rental_time = record.rental_time.strftime("%Y-%m-%d %H:%M:%S")
                    return_time = record.return_time.strftime("%Y-%m-%d %H:%M:%S") if record.return_time else "Not returned yet"
                    print(f"Car: {record.car.make} {record.car.model}, Plate Number: {record.plate_number}, Rental Time: {rental_time}, Return Time: {return_time}")
        else:
            print("Please log in first.")

car_rental_system = CarRentalSystem()
import getpass

while True:
    print("\n1. Register a new user")
    print("2. Log in")
    print("3. Quit")

    choice = input("Enter your choice (1-3): ")

    if choice == "1":
        username = input("Enter your username: ")
        password = getpass.getpass("Enter your password: ")  
        car_rental_system.register_user(username, password)
    elif choice == "2":
        username = input("Enter your username: ")
        password = getpass.getpass("Enter your password: ")  # Change to input to display it
        car_rental_system.login(username, password)
        if car_rental_system.current_user:
            print(f"Logged in as {car_rental_system.current_user.username}.")
            break
    elif choice == "3":
        print("Exiting the car rental system. Goodbye!")
        break
    else:
        print("Invalid choice. Please enter a number between 1 and 3.")

car1 = Car("Ford", "Mustang", "261REM")
car2 = Car("Toyota", "Camry", "307OJH")
car3 = Car("Honda", "Accord", "430QZY")

car_rental_system.add_car(car1)
car_rental_system.add_car(car2)
car_rental_system.add_car(car3)

while car_rental_system.current_user:
    print("\n1. Display available cars")
    print("2. Rent a car")
    print("3. Return a car")
    print("4. Rental history")
    print("5. Log out")

    choice = input("Enter your choice (1-5): ")

    if choice == "1":
        car_rental_system.display_available_cars()
    elif choice == "2":
        plate_number = input("Enter the plate number of the car you want to rent: ")
        
        # Prompt for rental duration with validation
        while True:
            rental_duration_input = input("Enter the rental duration in hours (6 hours = 500, 12 hours = 1000, or 24 hours = 2000): ")
            if rental_duration_input in ["6", "12", "24"]:
                rental_hours = int(rental_duration_input)
                break
            else:
                print("Invalid input. Please enter 6 hours = 500 , 12 hours = 1000, or 24 hours = 2000.")

        car_rental_system.rent_car(plate_number, rental_hours)
    elif choice == "3":
        plate_number = input("Enter the plate number of the car you want to return: ")
        car_rental_system.return_car(plate_number)
    elif choice == "4":
        car_rental_system.display_rental_history()
    elif choice == "5":
        print(f"Logging out {car_rental_system.current_user.username}.")
        car_rental_system.current_user = None
    else:
        print("Invalid choice. Please enter a number between 1 and 5.")
