# Program Final Assignment 

Introduction

Smart Campus Facility Booking System
This is a program about a Smart Campus Facility Booking System that we designed to help students and staff book campus facilities like study rooms, sports courts and event halls in an easy and organized way. Our program allows users to create an account, log in and reserve facilities based on their access type (Standard or Premium), while making sure all bookings follow rules such as time availability, capacity limits and any required fees. Users can also view, update or cancel their bookings and upgrade their access when needed. In addition, our program includes an admin side where facilities can be managed, availability can be updated and bookings can be monitored. Overall, our program focuses on making the booking process simple, clear and efficient for everyone.




UML Class Diagram:
https://drive.google.com/file/d/1QPYzgggCoKK1MZrLTMEn1o1bdxwssEBm/view?usp=sharing




The UML class diagram shows how all the classes in the system are connected and how they work together. It shows the relationships between the classes and how data moves in the system. First, there is inheritance between User, Student, and Administrator. The Student and Administrator both come from the User class, so they share the same basic information like name, email, and password. This is done so we don’t repeat the same things again and again. But still they are different, because student books and admin manage.
There is also an association between Student and Booking. This means the student can make bookings. One student can make many bookings, so one-to-many. And each booking belongs to only one student. Also there is another association between Facility and Booking. This means each booking is for one facility, but one facility can have many bookings. So again it is one-to-many. So booking is connected to both student and facility.
There is aggregation between BookingSystem and users, facilities, and bookings. This means the system has all of them inside it and manages them. The system controls everything and connects everything together. But still these objects are separate.
There is also dependency between BookingGUI and BookingSystem. The GUI depends on the system. The GUI only shows things and takes input, but the system does the real work.
Some assumptions are made. One student can have many bookings, but each booking is only for one student and one facility. One facility can have many bookings but not at the same time. Standard users can book less things, and premium users can book more things. Admin does not book, admin only manages. Also data is saved using pickle files so it stays.
Overall, the UML shows all relationships like inheritance, association, aggregation, and dependency, and it shows how everything is connected in a simple way.




The code:

import os
import pickle
import tkinter as tk
from tkinter import ttk, messagebox


# =========================================================
# Custom Errors
# =========================================================

class BookingError(Exception):
    pass


class LoginError(Exception):
    pass


class ValidationError(Exception):
    pass


# =========================================================
# Main Classes
# =========================================================

class User:
    def __init__(self, user_id, name, email, password):
        self.__user_id = user_id
        self.__name = name
        self.__email = email
        self.__password = password

    def get_user_id(self):
        return self.__user_id

    def get_name(self):
        return self.__name

    def set_name(self, name):
        if name.strip() == "":
            raise ValidationError("Name cannot be empty.")
        self.__name = name

    def get_email(self):
        return self.__email

    def set_email(self, email):
        if "@" not in email:
            raise ValidationError("Email must contain @.")
        self.__email = email

    def check_password(self, password):
        return self.__password == password

    def get_role(self):
        return "User"

    def get_access_type(self):
        return "Not Applicable"


class Student(User):
    def __init__(self, user_id, name, email, password, access_type):
        super().__init__(user_id, name, email, password)
        self.__access_type = access_type

    def get_access_type(self):
        return self.__access_type

    def set_access_type(self, access_type):
        if access_type != "Standard" and access_type != "Premium":
            raise ValidationError("Access type must be Standard or Premium.")
        self.__access_type = access_type

    def get_role(self):
        return "Student"


class Administrator(User):
    def __init__(self, user_id, name, email, password):
        super().__init__(user_id, name, email, password)

    def get_role(self):
        return "Administrator"


class Facility:
    def __init__(self, facility_id, name, facility_type, location, capacity, price_per_hour, time_slots):
        self.__facility_id = facility_id
        self.__name = name
        self.__facility_type = facility_type
        self.__location = location
        self.__capacity = capacity
        self.__price_per_hour = price_per_hour
        self.__time_slots = time_slots
        self.__status = "Available"

    def get_facility_id(self):
        return self.__facility_id

    def get_name(self):
        return self.__name

    def get_facility_type(self):
        return self.__facility_type

    def get_location(self):
        return self.__location

    def get_capacity(self):
        return self.__capacity

    def get_price_per_hour(self):
        return self.__price_per_hour

    def get_time_slots(self):
        return self.__time_slots

    def get_status(self):
        return self.__status

    def set_status(self, status):
        if status != "Available" and status != "Unavailable":
            raise ValidationError("Status must be Available or Unavailable.")
        self.__status = status

    def calculate_cost(self, hours):
        return self.__price_per_hour * hours

    def get_required_access(self):
        if self.__facility_type == "Study Room":
            return "Standard"
        else:
            return "Premium"


class Booking:
    def __init__(self, booking_id, student_id, facility_id, date, time_slot, hours, number_of_people, total_cost):
        self.__booking_id = booking_id
        self.__student_id = student_id
        self.__facility_id = facility_id
        self.__date = date
        self.__time_slot = time_slot
        self.__hours = hours
        self.__number_of_people = number_of_people
        self.__total_cost = total_cost
        self.__status = "Confirmed"

        if total_cost == 0:
            self.__payment_status = "Free"
        else:
            self.__payment_status = "Paid"

    def get_booking_id(self):
        return self.__booking_id

    def get_student_id(self):
        return self.__student_id

    def get_facility_id(self):
        return self.__facility_id

    def get_date(self):
        return self.__date

    def get_time_slot(self):
        return self.__time_slot

    def get_hours(self):
        return self.__hours

    def get_number_of_people(self):
        return self.__number_of_people

    def get_total_cost(self):
        return self.__total_cost

    def get_status(self):
        return self.__status

    def get_payment_status(self):
        return self.__payment_status

    def cancel_booking(self):
        self.__status = "Cancelled"

    def update_booking(self, date, time_slot, hours, number_of_people, total_cost):
        self.__date = date
        self.__time_slot = time_slot
        self.__hours = hours
        self.__number_of_people = number_of_people
        self.__total_cost = total_cost

        if total_cost == 0:
            self.__payment_status = "Free"
        else:
            self.__payment_status = "Paid"


# =========================================================
# Pickle Storage
# =========================================================

class Storage:
    def __init__(self):
        self.__folder = "smart_campus_final_data"

        self.__users_file = os.path.join(self.__folder, "users.pkl")
        self.__facilities_file = os.path.join(self.__folder, "facilities.pkl")
        self.__bookings_file = os.path.join(self.__folder, "bookings.pkl")

        if not os.path.exists(self.__folder):
            os.makedirs(self.__folder)

    def save_data(self, users, facilities, bookings):
        with open(self.__users_file, "wb") as file:
            pickle.dump(users, file)

        with open(self.__facilities_file, "wb") as file:
            pickle.dump(facilities, file)

        with open(self.__bookings_file, "wb") as file:
            pickle.dump(bookings, file)

    def load_file(self, file_name, default_value):
        if os.path.exists(file_name):
            with open(file_name, "rb") as file:
                return pickle.load(file)

        return default_value

    def load_data(self):
        users = self.load_file(self.__users_file, {})
        facilities = self.load_file(self.__facilities_file, {})
        bookings = self.load_file(self.__bookings_file, [])

        return users, facilities, bookings


# =========================================================
# System Logic
# =========================================================

class BookingSystem:
    def __init__(self):
        self.__storage = Storage()
        self.__users, self.__facilities, self.__bookings = self.__storage.load_data()
        self.__current_user = None

        self.create_default_data()

    def create_default_data(self):
        if len(self.__users) == 0:
            self.__users["admin"] = Administrator("admin", "System Admin", "admin@zu.ac.ae", "admin123")
            self.__users["S100"] = Student("S100", "Noura Student", "noura@zu.ac.ae", "1234", "Standard")
            self.__users["S200"] = Student("S200", "Ahmed Student", "ahmed@zu.ac.ae", "1234", "Premium")

        if len(self.__facilities) == 0:
            self.__facilities["F1"] = Facility(
                "F1", "Study Room A", "Study Room", "Building C, Floor 2", 6, 0,
                ["9:00-11:00", "11:00-1:00", "2:00-4:00"]
            )

            self.__facilities["F2"] = Facility(
                "F2", "Study Room B", "Study Room", "Library Wing", 4, 0,
                ["10:00-12:00", "1:00-3:00"]
            )

            self.__facilities["F3"] = Facility(
                "F3", "Basketball Court", "Sports Court", "Outdoor Area", 10, 20,
                ["8:00-10:00", "4:00-6:00"]
            )

            self.__facilities["F4"] = Facility(
                "F4", "Tennis Court", "Sports Court", "Outdoor Area", 4, 15,
                ["9:00-11:00", "5:00-7:00"]
            )

            self.__facilities["F5"] = Facility(
                "F5", "Event Hall", "Event Hall", "Main Building", 50, 100,
                ["10:00-12:00", "2:00-4:00", "4:00-6:00"]
            )

        self.save_all_data()

    def save_all_data(self):
        self.__storage.save_data(self.__users, self.__facilities, self.__bookings)

    def calculate_hours_from_time_slot(self, time_slot):
        parts = time_slot.split("-")

        start_part = parts[0]
        end_part = parts[1]

        start_hour = int(start_part.split(":")[0])
        end_hour = int(end_part.split(":")[0])

        if end_hour <= start_hour:
            end_hour = end_hour + 12

        return end_hour - start_hour

    def register_user(self, user_id, name, email, password, user_type, access_type):
        if user_id.strip() == "" or name.strip() == "" or email.strip() == "" or password.strip() == "":
            raise ValidationError("Please fill all fields.")

        if user_id in self.__users:
            raise ValidationError("This user ID already exists.")

        if "@" not in email:
            raise ValidationError("Please enter a valid email.")

        if user_type == "Student":
            self.__users[user_id] = Student(user_id, name, email, password, access_type)

        elif user_type == "Administrator":
            self.__users[user_id] = Administrator(user_id, name, email, password)

        else:
            raise ValidationError("Please choose a valid user type.")

        self.save_all_data()

    def login(self, user_id, password):
        if user_id not in self.__users:
            raise LoginError("User ID was not found.")

        if self.__users[user_id].check_password(password) == False:
            raise LoginError("Password is incorrect.")

        self.__current_user = self.__users[user_id]
        return self.__current_user

    def get_current_user(self):
        return self.__current_user

    def get_all_users(self):
        return self.__users

    def get_all_facilities(self):
        return self.__facilities

    def get_all_bookings(self):
        return self.__bookings

    def get_facility(self, facility_id):
        return self.__facilities[facility_id]

    def update_current_user_profile(self, name, email):
        if self.__current_user is None:
            raise ValidationError("No user is logged in.")

        self.__current_user.set_name(name)
        self.__current_user.set_email(email)
        self.save_all_data()

    def upgrade_current_user_access(self):
        if self.__current_user is None:
            raise ValidationError("No user is logged in.")

        if self.__current_user.get_role() != "Student":
            raise ValidationError("Only students can upgrade access.")

        if self.__current_user.get_access_type() == "Premium":
            raise ValidationError("You already have Premium access.")

        self.__current_user.set_access_type("Premium")
        self.save_all_data()

    def delete_user(self, user_id):
        user_id = str(user_id)

        if user_id not in self.__users:
            raise ValidationError("User was not found.")

        if user_id == "admin":
            raise ValidationError("The main admin account cannot be deleted.")

        del self.__users[user_id]
        self.save_all_data()

    def upgrade_access(self, user_id):
        user_id = str(user_id)

        if user_id not in self.__users:
            raise ValidationError("User was not found.")

        user = self.__users[user_id]

        if user.get_role() != "Student":
            raise ValidationError("Only student accounts can be upgraded.")

        if user.get_access_type() == "Premium":
            raise ValidationError("This student already has Premium access.")

        user.set_access_type("Premium")
        self.save_all_data()

    def check_access_permission(self, student, facility):
        if student.get_access_type() == "Standard" and facility.get_required_access() == "Premium":
            raise BookingError("This facility requires Premium access. Please upgrade to book it.")

    def is_time_slot_taken(self, facility_id, date, time_slot, ignored_booking_id=""):
        for booking in self.__bookings:
            same_booking = booking.get_booking_id() == ignored_booking_id
            same_facility = booking.get_facility_id() == facility_id
            same_date = booking.get_date() == date
            same_time = booking.get_time_slot() == time_slot
            active_booking = booking.get_status() == "Confirmed"

            if same_booking == False and same_facility and same_date and same_time and active_booking:
                return True

        return False

    def create_booking(self, facility_id, date, time_slot, number_of_people):
        if self.__current_user is None:
            raise BookingError("Please login first.")

        if self.__current_user.get_role() != "Student":
            raise BookingError("Only students can create bookings.")

        if facility_id not in self.__facilities:
            raise BookingError("Facility was not found.")

        facility = self.__facilities[facility_id]
        student = self.__current_user

        if facility.get_status() != "Available":
            raise BookingError("This facility is currently unavailable.")

        self.check_access_permission(student, facility)

        if date.strip() == "" or time_slot.strip() == "":
            raise ValidationError("Date and time cannot be empty.")

        if number_of_people <= 0:
            raise ValidationError("Number of people must be more than 0.")

        if number_of_people > facility.get_capacity():
            raise BookingError("Number of people cannot be more than the facility capacity.")

        if self.is_time_slot_taken(facility_id, date, time_slot):
            raise BookingError("This facility is already booked at this date and time.")

        hours = self.calculate_hours_from_time_slot(time_slot)
        total_cost = facility.calculate_cost(hours)

        booking_id = "B" + str(len(self.__bookings) + 1)

        booking = Booking(
            booking_id,
            student.get_user_id(),
            facility_id,
            date,
            time_slot,
            hours,
            number_of_people,
            total_cost
        )

        self.__bookings.append(booking)
        self.save_all_data()

        return booking

    def modify_booking(self, booking_id, date, time_slot, number_of_people):
        for booking in self.__bookings:
            if booking.get_booking_id() == booking_id:
                facility = self.get_facility(booking.get_facility_id())

                if booking.get_status() != "Confirmed":
                    raise BookingError("Cancelled bookings cannot be modified.")

                if number_of_people <= 0:
                    raise ValidationError("Number of people must be more than 0.")

                if number_of_people > facility.get_capacity():
                    raise BookingError("Number of people cannot be more than the facility capacity.")

                if self.is_time_slot_taken(booking.get_facility_id(), date, time_slot, booking_id):
                    raise BookingError("This facility is already booked at this date and time.")

                hours = self.calculate_hours_from_time_slot(time_slot)
                total_cost = facility.calculate_cost(hours)

                booking.update_booking(date, time_slot, hours, number_of_people, total_cost)

                self.save_all_data()
                return booking

        raise BookingError("Booking was not found.")

    def get_my_bookings(self):
        my_bookings = []

        for booking in self.__bookings:
            if booking.get_student_id() == self.__current_user.get_user_id():
                my_bookings.append(booking)

        return my_bookings

    def cancel_booking(self, booking_id):
        for booking in self.__bookings:
            if booking.get_booking_id() == booking_id:
                booking.cancel_booking()
                self.save_all_data()
                return

        raise BookingError("Booking was not found.")

    def update_facility_status(self, facility_id, status):
        if facility_id not in self.__facilities:
            raise ValidationError("Facility was not found.")

        self.__facilities[facility_id].set_status(status)
        self.save_all_data()

    def booking_activity_per_day(self):
        activity = {}

        for booking in self.__bookings:
            if booking.get_status() == "Confirmed":
                date = booking.get_date()

                if date not in activity:
                    activity[date] = 0

                activity[date] = activity[date] + 1

        return activity

    def facility_usage(self):
        usage = {}

        for facility_id in self.__facilities:
            usage[facility_id] = 0

        for booking in self.__bookings:
            if booking.get_status() == "Confirmed":
                facility_id = booking.get_facility_id()
                usage[facility_id] = usage[facility_id] + 1

        return usage


# =========================================================
# GUI
# =========================================================

class BookingGUI:
    def __init__(self):
        self.system = BookingSystem()

        self.root = tk.Tk()
        self.root.title("Smart Campus Facility Booking System")
        self.root.geometry("1200x700")
        self.root.resizable(False, False)

        self.show_login_screen()

    def clear_screen(self):
        for widget in self.root.winfo_children():
            widget.destroy()

    def show_title(self, title):
        tk.Label(self.root, text=title, font=("Arial", 20, "bold")).pack(pady=15)

    def get_facility_id_from_choice(self, choice):
        return choice.split("] ")[1].split(" - ")[0]

    def show_login_screen(self):
        self.clear_screen()
        self.show_title("Smart Campus Facility Booking System")

        tk.Label(
            self.root,
            text="Welcome to the Smart Campus booking system. Please log in or create an account to continue.",
            font=("Arial", 10),
            wraplength=900,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        frame = tk.Frame(self.root)
        frame.pack(pady=20)

        tk.Label(frame, text="User ID:").grid(row=0, column=0, pady=5, sticky="w")
        user_id_entry = tk.Entry(frame, width=30)
        user_id_entry.grid(row=0, column=1, pady=5)

        tk.Label(frame, text="Password:").grid(row=1, column=0, pady=5, sticky="w")
        password_entry = tk.Entry(frame, width=30, show="*")
        password_entry.grid(row=1, column=1, pady=5)

        def login_action():
            try:
                user = self.system.login(user_id_entry.get(), password_entry.get())

                if user.get_role() == "Administrator":
                    self.show_admin_screen()
                else:
                    self.show_student_screen()

            except LoginError as error:
                messagebox.showerror("Login Error", str(error))

        tk.Button(frame, text="Login", width=22, command=login_action).grid(row=2, column=0, columnspan=2, pady=10)
        tk.Button(frame, text="Create Account", width=22, command=self.show_register_screen).grid(row=3, column=0, columnspan=2)

        tk.Label(
            self.root,
            text="Demo accounts:\nAdmin: admin / admin123\nStandard Student: S100 / 1234\nPremium Student: S200 / 1234",
            fg="gray"
        ).pack(pady=20)

    def show_register_screen(self):
        self.clear_screen()
        self.show_title("Create Account")

        tk.Label(
            self.root,
            text="Create a new account by entering your details. Students can choose Standard or Premium access.",
            font=("Arial", 10),
            wraplength=900,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        frame = tk.Frame(self.root)
        frame.pack(pady=20)

        labels = ["User ID:", "Name:", "Email:", "Password:"]
        entries = []

        for i in range(len(labels)):
            tk.Label(frame, text=labels[i]).grid(row=i, column=0, pady=5, sticky="w")
            entry = tk.Entry(frame, width=30)
            entry.grid(row=i, column=1, pady=5)
            entries.append(entry)

        tk.Label(frame, text="User Type:").grid(row=4, column=0, pady=5, sticky="w")
        user_type_box = ttk.Combobox(frame, values=["Student", "Administrator"], state="readonly", width=27)
        user_type_box.set("Student")
        user_type_box.grid(row=4, column=1, pady=5)

        tk.Label(frame, text="Access Type:").grid(row=5, column=0, pady=5, sticky="w")
        access_box = ttk.Combobox(frame, values=["Standard", "Premium"], state="readonly", width=27)
        access_box.set("Standard")
        access_box.grid(row=5, column=1, pady=5)

        def register_action():
            try:
                self.system.register_user(
                    entries[0].get(),
                    entries[1].get(),
                    entries[2].get(),
                    entries[3].get(),
                    user_type_box.get(),
                    access_box.get()
                )

                messagebox.showinfo("Success", "Account created successfully.")
                self.show_login_screen()

            except ValidationError as error:
                messagebox.showerror("Register Error", str(error))

        tk.Button(frame, text="Create Account", width=20, command=register_action).grid(row=6, column=0, columnspan=2, pady=10)
        tk.Button(frame, text="Back", width=20, command=self.show_login_screen).grid(row=7, column=0, columnspan=2)

    def show_student_screen(self):
        self.clear_screen()

        user = self.system.get_current_user()
        self.show_title("Student Dashboard")

        tk.Label(
            self.root,
            text="Welcome, " + user.get_name() + "!",
            font=("Arial", 14, "bold")
        ).pack(pady=5)

        tk.Label(
            self.root,
            text="Role: " + user.get_role() + "   |   Access Type: " + user.get_access_type(),
            font=("Arial", 11)
        ).pack(pady=2)

        tk.Label(
            self.root,
            text="Use this dashboard to view campus facilities, make a booking, manage your bookings, and update your personal details.",
            font=("Arial", 10),
            wraplength=900,
            justify="center",
            fg="gray30"
        ).pack(pady=8)

        buttons_frame = tk.Frame(self.root)
        buttons_frame.pack(pady=15)

        tk.Button(buttons_frame, text="View Facilities", width=30, command=self.show_facilities_screen).pack(pady=5)
        tk.Label(buttons_frame, text="See all facilities, their access level, capacity, price, and time slots.", fg="gray35").pack()

        tk.Button(buttons_frame, text="Book Facility", width=30, command=self.show_booking_screen).pack(pady=5)
        tk.Label(buttons_frame, text="Choose a facility, date, and time slot to make a new booking.", fg="gray35").pack()

        tk.Button(buttons_frame, text="My Bookings", width=30, command=self.show_my_bookings_screen).pack(pady=5)
        tk.Label(buttons_frame, text="View, modify, or cancel your existing bookings.", fg="gray35").pack()

        tk.Button(buttons_frame, text="View / Update My Details", width=30, command=self.show_profile_screen).pack(pady=5)
        tk.Label(buttons_frame, text="Update your name, email, and check your access type.", fg="gray35").pack()

        tk.Button(buttons_frame, text="Logout", width=30, command=self.show_login_screen).pack(pady=15)

    def show_profile_screen(self):
        self.clear_screen()
        self.show_title("My User Details")

        user = self.system.get_current_user()

        tk.Label(
            self.root,
            text="This page shows your account details. You can edit your name and email, or upgrade to Premium if you are currently Standard.",
            font=("Arial", 10),
            wraplength=900,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        frame = tk.Frame(self.root)
        frame.pack(pady=20)

        tk.Label(frame, text="User ID:").grid(row=0, column=0, pady=5, sticky="w")
        tk.Label(frame, text=user.get_user_id()).grid(row=0, column=1, pady=5, sticky="w")

        tk.Label(frame, text="Role:").grid(row=1, column=0, pady=5, sticky="w")
        tk.Label(frame, text=user.get_role()).grid(row=1, column=1, pady=5, sticky="w")

        tk.Label(frame, text="Access Type:").grid(row=2, column=0, pady=5, sticky="w")
        tk.Label(frame, text=user.get_access_type()).grid(row=2, column=1, pady=5, sticky="w")

        tk.Label(frame, text="Name:").grid(row=3, column=0, pady=5, sticky="w")
        name_entry = tk.Entry(frame, width=30)
        name_entry.insert(0, user.get_name())
        name_entry.grid(row=3, column=1, pady=5)

        tk.Label(frame, text="Email:").grid(row=4, column=0, pady=5, sticky="w")
        email_entry = tk.Entry(frame, width=30)
        email_entry.insert(0, user.get_email())
        email_entry.grid(row=4, column=1, pady=5)

        def update_action():
            try:
                self.system.update_current_user_profile(name_entry.get(), email_entry.get())
                messagebox.showinfo("Updated", "Your details were updated.")
                self.show_student_screen()

            except ValidationError as error:
                messagebox.showerror("Error", str(error))

        def upgrade_action():
            try:
                self.system.upgrade_current_user_access()
                messagebox.showinfo("Upgraded", "Your access type is now Premium.")
                self.show_profile_screen()

            except ValidationError as error:
                messagebox.showerror("Error", str(error))

        tk.Button(frame, text="Save Changes", width=22, command=update_action).grid(row=5, column=0, columnspan=2, pady=5)

        if user.get_access_type() == "Premium":
            tk.Button(frame, text="Already Premium", width=22, state="disabled").grid(row=6, column=0, columnspan=2, pady=5)
        else:
            tk.Button(frame, text="Upgrade to Premium", width=22, command=upgrade_action).grid(row=6, column=0, columnspan=2, pady=5)

        tk.Button(frame, text="Back", width=22, command=self.show_student_screen).grid(row=7, column=0, columnspan=2, pady=5)

    def show_facilities_screen(self):
        self.clear_screen()
        self.show_title("Facilities")

        tk.Label(
            self.root,
            text="Below is the list of all facilities in the system. You can see the required access level, type, location, capacity, price, status, and available time slots.",
            font=("Arial", 10),
            wraplength=1000,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        tk.Label(
            self.root,
            text="Standard users can view all facilities, but they can only book Standard facilities. Premium users can book both Standard and Premium facilities.",
            font=("Arial", 10),
            wraplength=1000,
            justify="center",
            fg="gray35"
        ).pack(pady=2)

        table_frame = tk.Frame(self.root)
        table_frame.pack(pady=10)

        columns = ("Access", "ID", "Name", "Type", "Location", "Capacity", "Price", "Status", "Time Slots")

        tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings",
            height=12
        )

        for column in columns:
            tree.heading(column, text=column)

            if column == "Time Slots":
                tree.column(column, width=250)
            elif column == "Location":
                tree.column(column, width=140)
            elif column == "Name":
                tree.column(column, width=140)
            elif column == "Access":
                tree.column(column, width=100)
            elif column == "Price":
                tree.column(column, width=120)
            else:
                tree.column(column, width=90)

        for facility_id, facility in self.system.get_all_facilities().items():
            tree.insert("", "end", values=(
                facility.get_required_access(),
                facility_id,
                facility.get_name(),
                facility.get_facility_type(),
                facility.get_location(),
                facility.get_capacity(),
                str(facility.get_price_per_hour()) + " AED/hour",
                facility.get_status(),
                ", ".join(facility.get_time_slots())
            ))

        tree.pack()

        back_command = self.show_student_screen

        if self.system.get_current_user().get_role() == "Administrator":
            back_command = self.show_admin_screen

        tk.Label(
            self.root,
            text="Tip: Check the Access column before booking a facility.",
            font=("Arial", 10, "italic"),
            fg="gray35"
        ).pack(pady=5)

        tk.Button(self.root, text="Back", width=20, command=back_command).pack(pady=10)

    def show_booking_screen(self):
        self.clear_screen()
        self.show_title("Book a Facility")

        tk.Label(
            self.root,
            text="Select a facility, date, and time slot. The system will automatically calculate the duration and total cost from the selected time slot.",
            font=("Arial", 10),
            wraplength=900,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        frame = tk.Frame(self.root)
        frame.pack(pady=10)

        tk.Label(frame, text="Facility:").grid(row=0, column=0, pady=5, sticky="w")

        facility_choices = []

        for facility_id, facility in self.system.get_all_facilities().items():
            choice = "[" + facility.get_required_access().upper() + "] " + facility_id + " - " + facility.get_name()
            facility_choices.append(choice)

        facility_box = ttk.Combobox(frame, values=facility_choices, state="readonly", width=38)
        facility_box.grid(row=0, column=1, pady=5)

        if len(facility_choices) > 0:
            facility_box.set(facility_choices[0])

        tk.Label(frame, text="Date:").grid(row=1, column=0, pady=5, sticky="w")

        date_choices = [
            "15 March 2026",
            "16 March 2026",
            "17 March 2026",
            "18 March 2026",
            "19 March 2026"
        ]

        date_box = ttk.Combobox(frame, values=date_choices, state="readonly", width=38)
        date_box.grid(row=1, column=1, pady=5)
        date_box.set(date_choices[0])

        tk.Label(frame, text="Time Slot:").grid(row=2, column=0, pady=5, sticky="w")
        time_box = ttk.Combobox(frame, state="readonly", width=38)
        time_box.grid(row=2, column=1, pady=5)

        duration_label = tk.Label(frame, text="Duration: 0 hour(s)", font=("Arial", 11, "bold"))
        duration_label.grid(row=3, column=0, columnspan=2, pady=5)

        cost_label = tk.Label(frame, text="Total Cost: 0 AED", font=("Arial", 11, "bold"))
        cost_label.grid(row=4, column=0, columnspan=2, pady=5)

        tk.Label(frame, text="Number of People:").grid(row=5, column=0, pady=5, sticky="w")
        people_entry = tk.Entry(frame, width=41)
        people_entry.grid(row=5, column=1, pady=5)

        def update_cost():
            try:
                facility_id = self.get_facility_id_from_choice(facility_box.get())
                facility = self.system.get_facility(facility_id)
                time_slot = time_box.get()

                hours = self.system.calculate_hours_from_time_slot(time_slot)
                total_cost = facility.calculate_cost(hours)

                duration_label.config(text="Duration: " + str(hours) + " hour(s)")
                cost_label.config(text="Total Cost: " + str(total_cost) + " AED")

            except:
                duration_label.config(text="Duration: 0 hour(s)")
                cost_label.config(text="Total Cost: 0 AED")

        def update_time_slots(event=None):
            facility_id = self.get_facility_id_from_choice(facility_box.get())
            facility = self.system.get_facility(facility_id)

            time_box["values"] = facility.get_time_slots()

            if len(facility.get_time_slots()) > 0:
                time_box.set(facility.get_time_slots()[0])

            update_cost()

        facility_box.bind("<<ComboboxSelected>>", update_time_slots)
        time_box.bind("<<ComboboxSelected>>", lambda event: update_cost())

        update_time_slots()

        def book_action():
            try:
                facility_id = self.get_facility_id_from_choice(facility_box.get())
                number_of_people = int(people_entry.get())

                booking = self.system.create_booking(
                    facility_id,
                    date_box.get(),
                    time_box.get(),
                    number_of_people
                )

                facility = self.system.get_facility(facility_id)
                user = self.system.get_current_user()

                receipt = (
                    "SMART CAMPUS BOOKING RECEIPT\n"
                    "----------------------------------\n"
                    "Booking ID: " + booking.get_booking_id() + "\n"
                    "User Name: " + user.get_name() + "\n"
                    "User ID: " + user.get_user_id() + "\n"
                    "Access Type: " + user.get_access_type() + "\n\n"
                    "Facility: " + facility.get_name() + "\n"
                    "Required Access: " + facility.get_required_access() + "\n"
                    "Facility Type: " + facility.get_facility_type() + "\n"
                    "Location: " + facility.get_location() + "\n"
                    "Facility Capacity: " + str(facility.get_capacity()) + "\n\n"
                    "Date: " + booking.get_date() + "\n"
                    "Time Slot: " + booking.get_time_slot() + "\n"
                    "Duration: " + str(booking.get_hours()) + " hour(s)\n"
                    "Number of People: " + str(booking.get_number_of_people()) + "\n\n"
                    "Rate: " + str(facility.get_price_per_hour()) + " AED/hour\n"
                    "Total Cost: " + str(booking.get_total_cost()) + " AED\n"
                    "Payment Status: " + booking.get_payment_status() + "\n"
                    "Booking Status: " + booking.get_status()
                )

                messagebox.showinfo("Booking Receipt", receipt)
                self.show_student_screen()

            except ValueError:
                messagebox.showerror("Input Error", "Number of people must be a number.")

            except (BookingError, ValidationError) as error:
                messagebox.showerror("Booking Error", str(error))

        tk.Button(frame, text="Confirm Booking", width=22, command=book_action).grid(row=6, column=0, columnspan=2, pady=10)
        tk.Button(frame, text="Back", width=22, command=self.show_student_screen).grid(row=7, column=0, columnspan=2, pady=5)

    def show_my_bookings_screen(self):
        self.clear_screen()
        self.show_title("My Bookings")

        tk.Label(
            self.root,
            text="Here you can view all your bookings. Select a booking from the table if you want to modify it or cancel it.",
            font=("Arial", 10),
            wraplength=1000,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        table_frame = tk.Frame(self.root)
        table_frame.pack(pady=10)

        columns = ("Booking ID", "Facility ID", "Date", "Time", "Duration", "People", "Cost", "Status")

        tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings",
            height=12
        )

        for column in columns:
            tree.heading(column, text=column)
            tree.column(column, width=130)

        for booking in self.system.get_my_bookings():
            tree.insert("", "end", values=(
                booking.get_booking_id(),
                booking.get_facility_id(),
                booking.get_date(),
                booking.get_time_slot(),
                str(booking.get_hours()) + " hour(s)",
                booking.get_number_of_people(),
                str(booking.get_total_cost()) + " AED",
                booking.get_status()
            ))

        tree.pack()

        tk.Label(
            self.root,
            text="Please select one booking first, then choose Modify or Cancel.",
            font=("Arial", 10, "italic"),
            fg="gray35"
        ).pack(pady=5)

        def get_selected_booking_id():
            selected = tree.focus()

            if selected == "":
                messagebox.showerror("Error", "Please select a booking first.")
                return None

            return str(tree.item(selected)["values"][0])

        def cancel_selected():
            booking_id = get_selected_booking_id()

            if booking_id is None:
                return

            try:
                self.system.cancel_booking(booking_id)
                messagebox.showinfo("Cancelled", "Booking cancelled successfully.")
                self.show_my_bookings_screen()

            except BookingError as error:
                messagebox.showerror("Error", str(error))

        def modify_selected():
            booking_id = get_selected_booking_id()

            if booking_id is None:
                return

            self.show_modify_booking_screen(booking_id)

        tk.Button(self.root, text="Modify Selected Booking", width=25, command=modify_selected).pack(pady=5)
        tk.Button(self.root, text="Cancel Selected Booking", width=25, command=cancel_selected).pack(pady=5)
        tk.Button(self.root, text="Back", width=20, command=self.show_student_screen).pack(pady=5)

    def show_modify_booking_screen(self, booking_id):
        selected_booking = None

        for booking in self.system.get_my_bookings():
            if booking.get_booking_id() == booking_id:
                selected_booking = booking

        if selected_booking is None:
            messagebox.showerror("Error", "Booking was not found.")
            return

        self.clear_screen()
        self.show_title("Modify Booking")

        tk.Label(
            self.root,
            text="Change the date, time slot, or number of people for this booking. The duration and cost update automatically.",
            font=("Arial", 10),
            wraplength=900,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        facility = self.system.get_facility(selected_booking.get_facility_id())

        frame = tk.Frame(self.root)
        frame.pack(pady=10)

        tk.Label(frame, text="Booking ID:").grid(row=0, column=0, pady=5, sticky="w")
        tk.Label(frame, text=selected_booking.get_booking_id()).grid(row=0, column=1, pady=5, sticky="w")

        tk.Label(frame, text="Facility:").grid(row=1, column=0, pady=5, sticky="w")
        tk.Label(frame, text=facility.get_name()).grid(row=1, column=1, pady=5, sticky="w")

        tk.Label(frame, text="Date:").grid(row=2, column=0, pady=5, sticky="w")

        date_choices = [
            "15 March 2026",
            "16 March 2026",
            "17 March 2026",
            "18 March 2026",
            "19 March 2026"
        ]

        date_box = ttk.Combobox(frame, values=date_choices, state="readonly", width=35)
        date_box.grid(row=2, column=1, pady=5)
        date_box.set(selected_booking.get_date())

        tk.Label(frame, text="Time Slot:").grid(row=3, column=0, pady=5, sticky="w")
        time_box = ttk.Combobox(frame, values=facility.get_time_slots(), state="readonly", width=35)
        time_box.grid(row=3, column=1, pady=5)
        time_box.set(selected_booking.get_time_slot())

        duration_label = tk.Label(frame, text="", font=("Arial", 11, "bold"))
        duration_label.grid(row=4, column=0, columnspan=2, pady=5)

        cost_label = tk.Label(frame, text="", font=("Arial", 11, "bold"))
        cost_label.grid(row=5, column=0, columnspan=2, pady=5)

        tk.Label(frame, text="Number of People:").grid(row=6, column=0, pady=5, sticky="w")
        people_entry = tk.Entry(frame, width=38)
        people_entry.insert(0, str(selected_booking.get_number_of_people()))
        people_entry.grid(row=6, column=1, pady=5)

        def update_modify_cost(event=None):
            time_slot = time_box.get()
            hours = self.system.calculate_hours_from_time_slot(time_slot)
            total_cost = facility.calculate_cost(hours)

            duration_label.config(text="Duration: " + str(hours) + " hour(s)")
            cost_label.config(text="Total Cost: " + str(total_cost) + " AED")

        time_box.bind("<<ComboboxSelected>>", update_modify_cost)
        update_modify_cost()

        def save_changes():
            try:
                number_of_people = int(people_entry.get())

                self.system.modify_booking(
                    selected_booking.get_booking_id(),
                    date_box.get(),
                    time_box.get(),
                    number_of_people
                )

                messagebox.showinfo("Updated", "Booking updated successfully.")
                self.show_my_bookings_screen()

            except ValueError:
                messagebox.showerror("Input Error", "Number of people must be a number.")

            except (BookingError, ValidationError) as error:
                messagebox.showerror("Error", str(error))

        tk.Button(frame, text="Save Changes", width=22, command=save_changes).grid(row=7, column=0, columnspan=2, pady=10)
        tk.Button(frame, text="Back", width=22, command=self.show_my_bookings_screen).grid(row=8, column=0, columnspan=2)

    def show_admin_screen(self):
        self.clear_screen()
        self.show_title("Admin Dashboard")

        current_user = self.system.get_current_user()

        tk.Label(
            self.root,
            text="Welcome, " + current_user.get_name() + "!",
            font=("Arial", 14, "bold")
        ).pack(pady=5)

        tk.Label(
            self.root,
            text="Role: Administrator",
            font=("Arial", 11)
        ).pack(pady=2)

        tk.Label(
            self.root,
            text="Use this dashboard to manage users, monitor bookings, update facility availability, and review reports about facility usage.",
            font=("Arial", 10),
            wraplength=900,
            justify="center",
            fg="gray30"
        ).pack(pady=8)

        buttons_frame = tk.Frame(self.root)
        buttons_frame.pack(pady=20)

        tk.Button(buttons_frame, text="View Facilities", width=30, command=self.show_facilities_screen).pack(pady=5)
        tk.Label(buttons_frame, text="View all facilities, including access level, price, capacity, and time slots.", fg="gray35").pack()

        tk.Button(buttons_frame, text="View All Bookings", width=30, command=self.show_all_bookings_screen).pack(pady=5)
        tk.Label(buttons_frame, text="See all bookings made by students in the system.", fg="gray35").pack()

        tk.Button(buttons_frame, text="Manage Users", width=30, command=self.show_manage_users_screen).pack(pady=5)
        tk.Label(buttons_frame, text="View users, upgrade student access, or delete a selected user.", fg="gray35").pack()

        tk.Button(buttons_frame, text="Update Facility Status", width=30, command=self.show_update_status_screen).pack(pady=5)
        tk.Label(buttons_frame, text="Change a facility status between Available and Unavailable.", fg="gray35").pack()

        tk.Button(buttons_frame, text="Reports: Activity and Usage", width=30, command=self.show_reports_screen).pack(pady=5)
        tk.Label(buttons_frame, text="Check booking activity per day and facility usage summary.", fg="gray35").pack()

        tk.Button(buttons_frame, text="Logout", width=30, command=self.show_login_screen).pack(pady=15)

    def show_all_bookings_screen(self):
        self.clear_screen()
        self.show_title("All Bookings")

        tk.Label(
            self.root,
            text="This screen displays all bookings made in the system. It helps the administrator monitor booking records, dates, time slots, costs, and current booking status.",
            font=("Arial", 10),
            wraplength=1000,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        table_frame = tk.Frame(self.root)
        table_frame.pack(pady=10)

        columns = ("Booking ID", "Student ID", "Facility ID", "Date", "Time", "Duration", "People", "Cost", "Status")

        tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings",
            height=12
        )

        for column in columns:
            tree.heading(column, text=column)
            tree.column(column, width=120)

        for booking in self.system.get_all_bookings():
            tree.insert("", "end", values=(
                booking.get_booking_id(),
                booking.get_student_id(),
                booking.get_facility_id(),
                booking.get_date(),
                booking.get_time_slot(),
                str(booking.get_hours()) + " hour(s)",
                booking.get_number_of_people(),
                str(booking.get_total_cost()) + " AED",
                booking.get_status()
            ))

        tree.pack()

        tk.Label(
            self.root,
            text="Confirmed bookings are active. Cancelled bookings remain in the record for tracking purposes.",
            font=("Arial", 10, "italic"),
            fg="gray35"
        ).pack(pady=5)

        tk.Button(self.root, text="Back", width=20, command=self.show_admin_screen).pack(pady=10)

    def show_manage_users_screen(self):
        self.clear_screen()
        self.show_title("Manage Users")

        tk.Label(
            self.root,
            text="This screen shows all registered users. Select a user from the table, then upgrade a student to Premium or delete a selected user.",
            font=("Arial", 10),
            wraplength=1000,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        tk.Label(
            self.root,
            text="Note: Administrator accounts show 'Not Applicable' under Access Type because access type is only used for students.",
            font=("Arial", 10),
            wraplength=1000,
            justify="center",
            fg="gray35"
        ).pack(pady=2)

        table_frame = tk.Frame(self.root)
        table_frame.pack(pady=10)

        columns = ("User ID", "Name", "Email", "Role", "Access Type")

        tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings",
            height=12
        )

        for column in columns:
            tree.heading(column, text=column)
            tree.column(column, width=210)

        for user_id, user in self.system.get_all_users().items():
            tree.insert("", "end", values=(
                user_id,
                user.get_name(),
                user.get_email(),
                user.get_role(),
                user.get_access_type()
            ))

        tree.pack()

        tk.Label(
            self.root,
            text="Please select one row from the table before pressing Upgrade or Delete.",
            font=("Arial", 10, "italic"),
            fg="gray35"
        ).pack(pady=5)

        def get_selected_user_id():
            selected = tree.focus()

            if selected == "":
                messagebox.showerror("Error", "Please select a user first.")
                return None

            return str(tree.item(selected)["values"][0])

        def upgrade_selected():
            user_id = get_selected_user_id()

            if user_id is None:
                return

            try:
                self.system.upgrade_access(user_id)
                messagebox.showinfo("Upgraded", "Student access upgraded to Premium.")
                self.show_manage_users_screen()

            except ValidationError as error:
                messagebox.showerror("Error", str(error))

        def delete_selected():
            user_id = get_selected_user_id()

            if user_id is None:
                return

            try:
                self.system.delete_user(user_id)
                messagebox.showinfo("Deleted", "User deleted successfully.")
                self.show_manage_users_screen()

            except ValidationError as error:
                messagebox.showerror("Error", str(error))

        button_frame = tk.Frame(self.root)
        button_frame.pack(pady=10)

        tk.Button(button_frame, text="Upgrade Selected Student", width=25, command=upgrade_selected).pack(pady=5)
        tk.Button(button_frame, text="Delete Selected User", width=25, command=delete_selected).pack(pady=5)
        tk.Button(button_frame, text="Back", width=20, command=self.show_admin_screen).pack(pady=5)

    def show_update_status_screen(self):
        self.clear_screen()
        self.show_title("Update Facility Status")

        tk.Label(
            self.root,
            text="Use this screen to change whether a facility is available for booking or temporarily unavailable.",
            font=("Arial", 10),
            wraplength=900,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        frame = tk.Frame(self.root)
        frame.pack(pady=20)

        facility_choices = []

        for facility_id, facility in self.system.get_all_facilities().items():
            facility_choices.append(facility_id + " - " + facility.get_name())

        tk.Label(frame, text="Facility:").grid(row=0, column=0, pady=5, sticky="w")

        facility_box = ttk.Combobox(frame, values=facility_choices, state="readonly", width=35)
        facility_box.grid(row=0, column=1, pady=5)

        if len(facility_choices) > 0:
            facility_box.set(facility_choices[0])

        tk.Label(frame, text="Status:").grid(row=1, column=0, pady=5, sticky="w")

        status_box = ttk.Combobox(frame, values=["Available", "Unavailable"], state="readonly", width=35)
        status_box.set("Available")
        status_box.grid(row=1, column=1, pady=5)

        def update_action():
            try:
                facility_id = facility_box.get().split(" - ")[0]
                self.system.update_facility_status(facility_id, status_box.get())

                messagebox.showinfo("Updated", "Facility status updated successfully.")
                self.show_admin_screen()

            except ValidationError as error:
                messagebox.showerror("Error", str(error))

        tk.Button(frame, text="Update", width=22, command=update_action).grid(row=2, column=0, columnspan=2, pady=10)
        tk.Button(frame, text="Back", width=22, command=self.show_admin_screen).grid(row=3, column=0, columnspan=2)

    def show_reports_screen(self):
        self.clear_screen()
        self.show_title("Reports")

        tk.Label(
            self.root,
            text="This report summarizes daily booking activity and how often each facility has been used.",
            font=("Arial", 10),
            wraplength=900,
            justify="center",
            fg="gray30"
        ).pack(pady=5)

        text_box = tk.Text(self.root, width=120, height=25)
        text_box.pack(pady=10)

        text_box.insert(tk.END, "BOOKING ACTIVITY PER DAY\n")
        text_box.insert(tk.END, "------------------------\n")

        activity = self.system.booking_activity_per_day()

        if len(activity) == 0:
            text_box.insert(tk.END, "No confirmed bookings yet.\n")
        else:
            for date in activity:
                text_box.insert(tk.END, date + ": " + str(activity[date]) + " booking(s)\n")

        text_box.insert(tk.END, "\nFACILITY USAGE AND CAPACITY\n")
        text_box.insert(tk.END, "---------------------------\n")

        usage = self.system.facility_usage()

        for facility_id, count in usage.items():
            facility = self.system.get_facility(facility_id)

            line = (
                facility.get_name() +
                " | Required Access: " + facility.get_required_access() +
                " | Type: " + facility.get_facility_type() +
                " | Capacity: " + str(facility.get_capacity()) +
                " | Confirmed Bookings: " + str(count) +
                " | Status: " + facility.get_status() + "\n"
            )

            text_box.insert(tk.END, line)

        text_box.config(state="disabled")

        tk.Button(self.root, text="Back", width=20, command=self.show_admin_screen).pack(pady=10)

    def run(self):
        self.root.mainloop()


# =========================================================
# Run Program
# =========================================================

app = BookingGUI()
app.run()
















Graphical User Interface (GUI)
In our code, we implemented the GUI using tkinter. The GUI is simple and easy and easy to understand for the user. The GUI has many screens like login screen, create account screen, student dashboard, booking screen, my bookings screen, and admin dashboard. The user first logs in or creates an account. In creating an account, the user adds details like name, email, and password. So the system allows adding user details.
The user can also see their details and change their details in the profile screen. So the system also allows display and modify user details. The admin can also see all users and delete users or upgrade users. So we have to add, display, modify, and delete for users.
The GUI also lets the user see bookings, modify bookings, and cancel bookings. So the system also supports view, modify, and delete bookings. Everything is done through buttons and simple screens, so it is easy and clear.
Booking Interface
In the booking interface, the user can clearly see all facilities. The facilities include study rooms, sports courts, and event halls. Each facility shows its information like capacity, price per hour, time slots, and status. So the user can clearly see all details before booking.
The user selects a facility, then selects a date, and then selects a time slot. When the time slot is selected, the system automatically calculates the duration and also calculates the total cost. The cost is shown before confirming, so the user knows the cost before booking. Then the user confirms the booking.
After confirming, the system shows a receipt. The receipt shows booking ID, user name, facility name, date, time, number of people, cost, and status. So the system clearly shows all booking details.
Booking Constraints
The system checks many rules before making a booking. It checks the access type. If the user is Standard, they cannot book Premium facilities. So access is checked. It also checks if the facility is available or not. It checks the number of people and makes sure it does not go above the capacity. It also checks if the time slot is already taken or not.
So the system checks access, checks capacity, checks time slot, and checks availability. If something is wrong, it shows an error. So all bookings follow the rules and no wrong booking happens.
Admin Dashboard
The admin dashboard is also implemented in the GUI. The admin can see all bookings in the system. The admin can monitor bookings and see all details. The admin can also manage users, like upgrading a student from Standard to Premium, and also deleting users.
The admin can also update facility availability. The admin can change a facility from Available to Unavailable. The admin can also see reports. The reports show booking activity per day, and also show how many times each facility is used. So the admin can see usage and activity clearly.
Overall, the admin dashboard helps the admin manage everything, manage users, manage facilities, and monitor bookings in a simple and clear way.
Persistence (Pickle)
In our system, we used Pickle to save the data so the data does not get lost. The data is saved in files and not only in memory. So when the program closes, the data is still saved, and when we open the program again, the data comes back again. So the system keeps the data and does not lose the data.
We made separate files to store the data in a simple way. We have one file for users, one file for facilities, and one file for bookings. The users file stores all users like students and admins. The facilities file stores all facilities like study rooms, sports courts, and event halls. The bookings file stores all bookings that users make. So each file stores its own data and does not mix with other data.
We used pickle.dump() to save the data, and we used pickle.load() to load the data again. The system loads the data when it starts, and it saves the data again when something changes. So when we create a user, or make a booking, or update something, the data is saved again.
We used separate files so it is more clear and more organized. Each file has its own job, so it is easy to understand and easy to manage. Overall, Pickle helps us save the data, keep the data, and use the data again and again without losing it.

Error Handling
In our system, we handled errors by checking inputs and using different exceptions. We did not use only one error, we used different errors for different problems.
We used LoginError for wrong login, ValidationError for wrong input like empty fields or wrong email, and BookingError for booking problems like wrong access, full capacity, or taken time slot.
The system checks things again and again. It checks input, checks access, checks capacity, and checks time slot before booking. So it makes sure everything is correct.
We also used try and except in the GUI. When there is an error, a message shows to the user. So the user knows the problem.
Overall, the system checks errors, shows errors, and helps the user fix errors easily.












Test Screen shots:

In this page where a user signs up by entering their details like ID, name, email and password. It also lets them choose a user type, such as a student or an administrator and select whether they want standard or premium access.

Here it shows the user had created an account


This page where the user login.


If you are a student. This is the page where a welcome message is displayed along with the user and access types. It shows four main options to view facilities, book a facility, check my bookings or view/update my details and it also includes a logout button.




When you click on "View Facilities," it shows a complete list of everything available in the system. You can see a table that breaks down each facility by its access level, name and type, like study rooms or sports courts, along with its location and how many people it can hold. It also lists the price per hour in AED, the current status and the specific time slots available for booking.











In this is the "Book a Facility" page where you can actually reserve a spot by picking a facility, date and time slot from the dropdown menus. The system automatically calculates the total duration and cost based on what you choose and there is a box to type in the number of people coming with you. Once everything looks good, you just hit "Confirm Booking" to finish or "Back" to change something. And in the “Facility” dropdown it shows you if the facility is premium or standard so you can choose based on your access type.










When you click confirm it shows you the receipt with your booking information.


If you entered a number of people more than the room capacity it shows you this and tells you that number of people cannot be more than the facility capacity.



And if you try to select a premium facility it shows this which tells you that a premium access required and tells you to upgrade so you can book a premium facility.










In this you can see the "My Bookings" page, which lists all the reservations a user has made in a table format. This table shows important details like the booking and facility IDs, the date and time, how many people are going, the cost and the current status of the reservation. There are also buttons at the bottom to "Modify" or "Cancel" a selected booking if you need to make changes, plus a "Back" button to return to the previous screen.














When you click on the modify button, it lets you change the details of an existing reservation. On this page, you can update the date, the time slot or the number of people and the system automatically recalculates the duration and total cost for you. To finish, you just click "Save Changes" to update the booking or "Back" to leave the page.










When you click on “Save Changes” it shows you that the booking is updated.


When you click on the View / Update My Details button, it takes you to the "My User Details" page shown in this image. On this screen, you can see your User ID, role and current access type and it lets you edit your name and email address. There are also buttons to save your changes, upgrade your account to premium if you have standard access or just go back to the previous page.

And if you click on “Upgrade to Premium" it updates and shows you that your access type now is premium.













If you are an administrator. This page shows the Admin Dashboard for an administrator named Ahmed, starting with a welcome message and his role. The dashboard includes several buttons for managing the system, such as View Facilities to check details like access levels and prices and View All Bookings to see every reservation made by students. It also provides a Manage Users button to view or delete accounts, an Update Facility Status button to change availability, and a Reports: Activity and Usage button to review summary data. Finally, a Logout button is located at the bottom of the screen to sign out of the system.












When you click on the View Facilities button, it opens this page which shows a table containing the full list of facilities in the system. This table lets you see specific details for each facility, such as the required Access level, ID, Name, Type, Location and Capacity. You can also check the Price in AED per hour, the current Status and the Time Slots that are available for booking. There is a Back button at the bottom of the screen to return to the previous dashboard.












When you click on View All Bookings, it opens the All Bookings page shown in the screenshot. This screen displays a table of every reservation in the system, including columns for the Booking ID, Student ID, Facility ID and the specific Date and Time. It also shows the Duration, number of People, total Cost and the current Status for each entry to help administrators track records. At the bottom, there is a Back button to return to the previous menu.












When you click on Manage Users, it opens the page shown, which displays a list of all registered users in a table format. This table includes columns for the User ID, Name, Email, Role and Access Type. You can select a specific user from the list and then use the buttons at the bottom to Upgrade Selected Student to premium or Delete Selected User. There is also a Back button available to return to the previous screen.

When select a user and click “Delete Selected User” it delete it and show that the user are deleted successfully.



And if you select a user and click on “Upgrade Selected Student” it upgrades the user to premium.













When you click on the Update Facility Status button, it opens the screen shown, which allows the administrator to change whether a facility is available for booking or temporarily unavailable. On this page, you can select a specific Facility from the dropdown menu and then choose the desired Status. Once you have made your selection, you can click the Update button to apply the changes or use the Back button to return to the dashboard.


When you change the “Status” and click on upgrade it shows you that the status are updated.


When you click on the Reports: Activity and Usage button, it opens the Reports page, which provides a detailed summary of daily booking activity and facility usage. This screen is divided into two main sections: BOOKING ACTIVITY PER DAY, which lists specific dates and the number of bookings made on each and FACILITY USAGE AND CAPACITY, which breaks down each facility by its name, required access, type, capacity, number of confirmed bookings and current status. The page allows an administrator to quickly see which areas are most active, such as Study Room B with 3 confirmed bookings or check the status of larger venues like the Event Hall. A Back button is available at the bottom of the screen to return to the dashboard.








Test cases reflection :
The Smart Campus Facility Booking System effectively handles the essential tasks for both students and administrators. Our test cases show that the user profile section works well, letting students manage their own contact details while keeping their assigned access levels secure. The booking process is also very smooth, the system automatically calculates the total cost and hours whenever a change is made to a reservation. This makes it easy for students to modify their plans without any manual errors.
On the admin side, the dashboard provides a great overview of everything happening on campus. The ability to monitor every single booking and manage user accounts ensures that the system stays organized and secure. The reporting feature is especially helpful because it summarizes daily activity and shows which rooms are being used the most. Seeing that Study Room B is busier than the Event Hall, for example, helps in understanding facility demand. Overall, our tests confirm the system is reliable and user-friendly.
]









Summary:
Working on this facility booking system was a great way for us to see how programming actually works in the real world. Throughout the project, our main focus was on using Object-Oriented Programming (OOP) to make the code organized and easy to manage. By creating a main User class and then using inheritance for Students and Administrators, we were able to give different permissions to each person without rewriting the same code. This made the system much more efficient because a student can only book rooms, while an admin has the power to manage everything.
One of the most interesting parts was learning how to keep the data safe using the pickle module. In our earlier projects, everything would disappear when we closed the program but now, the booking history and user accounts are saved into .pkl files. This makes the app feel like a real piece of software that people can actually use day-to-day. We also spent a lot of time on error handling to make sure the program doesn't crash. For example, if a student tries to book a "Premium" room but they only have a "Standard" account, the system stops them and explains why.
The GUI part using Tkinter was also a big learning curve for the group. We had to think about the layout from a student's perspective to make sure the dashboard was simple and not confusing. Seeing the logic from the backend finally show up on a screen with buttons and labels was very satisfying. Overall, this assignment taught us how to connect different parts of Python, like logic, data storage and interface design into one working project. It really helped us understand how to build a complete application from scratch.
