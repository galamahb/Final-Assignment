# Program Final Assignment 

**Introduction**

Smart Campus Facility Booking System
This is a program about a Smart Campus Facility Booking System that we designed to help students and staff book campus facilities like study rooms, sports courts and event halls in an easy and organized way. Our program allows users to create an account, log in and reserve facilities based on their access type (Standard or Premium), while making sure all bookings follow rules such as time availability, capacity limits and any required fees. Users can also view, update or cancel their bookings and upgrade their access when needed. In addition, our program includes an admin side where facilities can be managed, availability can be updated and bookings can be monitored. Overall, our program focuses on making the booking process simple, clear and efficient for everyone.


-----------------------------------------------------------------------------------------------------------------------------------------------

**UML Class Diagram:**
https://drive.google.com/file/d/1QPYzgggCoKK1MZrLTMEn1o1bdxwssEBm/view?usp=sharing





The UML class diagram shows how all the classes in the system are connected and how they work together. It shows the relationships between the classes and how data moves in the system. First, there is inheritance between User, Student, and Administrator. The Student and Administrator both come from the User class, so they share the same basic information like name, email, and password. This is done so we don’t repeat the same things again and again. But still they are different, because student books and admin manage.
There is also an association between Student and Booking. This means the student can make bookings. One student can make many bookings, so one-to-many. And each booking belongs to only one student. Also there is another association between Facility and Booking. This means each booking is for one facility, but one facility can have many bookings. So again it is one-to-many. So booking is connected to both student and facility.
There is aggregation between BookingSystem and users, facilities, and bookings. This means the system has all of them inside it and manages them. The system controls everything and connects everything together. But still these objects are separate.
There is also dependency between BookingGUI and BookingSystem. The GUI depends on the system. The GUI only shows things and takes input, but the system does the real work.
Some assumptions are made. One student can have many bookings, but each booking is only for one student and one facility. One facility can have many bookings but not at the same time. Standard users can book less things, and premium users can book more things. Admin does not book, admin only manages. Also data is saved using pickle files so it stays.
Overall, the UML shows all relationships like inheritance, association, aggregation, and dependency, and it shows how everything is connected in a simple way.



-----------------------------------------------------------------------------------------------------------------------------------------------



**Graphical User Interface (GUI)**

In our code, we implemented the GUI using tkinter. The GUI is simple and easy and easy to understand for the user. The GUI has many screens like login screen, create account screen, student dashboard, booking screen, my bookings screen, and admin dashboard. The user first logs in or creates an account. In creating an account, the user adds details like name, email, and password. So the system allows adding user details.
The user can also see their details and change their details in the profile screen. So the system also allows display and modify user details. The admin can also see all users and delete users or upgrade users. So we have to add, display, modify, and delete for users.
The GUI also lets the user see bookings, modify bookings, and cancel bookings. So the system also supports view, modify, and delete bookings. Everything is done through buttons and simple screens, so it is easy and clear.

**Booking Interface**

In the booking interface, the user can clearly see all facilities. The facilities include study rooms, sports courts, and event halls. Each facility shows its information like capacity, price per hour, time slots, and status. So the user can clearly see all details before booking.
The user selects a facility, then selects a date, and then selects a time slot. When the time slot is selected, the system automatically calculates the duration and also calculates the total cost. The cost is shown before confirming, so the user knows the cost before booking. Then the user confirms the booking.
After confirming, the system shows a receipt. The receipt shows booking ID, user name, facility name, date, time, number of people, cost, and status. So the system clearly shows all booking details.

**Booking Constraints**

The system checks many rules before making a booking. It checks the access type. If the user is Standard, they cannot book Premium facilities. So access is checked. It also checks if the facility is available or not. It checks the number of people and makes sure it does not go above the capacity. It also checks if the time slot is already taken or not.
So the system checks access, checks capacity, checks time slot, and checks availability. If something is wrong, it shows an error. So all bookings follow the rules and no wrong booking happens.

**Admin Dashboard**

The admin dashboard is also implemented in the GUI. The admin can see all bookings in the system. The admin can monitor bookings and see all details. The admin can also manage users, like upgrading a student from Standard to Premium, and also deleting users.
The admin can also update facility availability. The admin can change a facility from Available to Unavailable. The admin can also see reports. The reports show booking activity per day, and also show how many times each facility is used. So the admin can see usage and activity clearly.
Overall, the admin dashboard helps the admin manage everything, manage users, manage facilities, and monitor bookings in a simple and clear way.

**Persistence (Pickle)**

In our system, we used Pickle to save the data so the data does not get lost. The data is saved in files and not only in memory. So when the program closes, the data is still saved, and when we open the program again, the data comes back again. So the system keeps the data and does not lose the data.
We made separate files to store the data in a simple way. We have one file for users, one file for facilities, and one file for bookings. The users file stores all users like students and admins. The facilities file stores all facilities like study rooms, sports courts, and event halls. The bookings file stores all bookings that users make. So each file stores its own data and does not mix with other data.
We used pickle.dump() to save the data, and we used pickle.load() to load the data again. The system loads the data when it starts, and it saves the data again when something changes. So when we create a user, or make a booking, or update something, the data is saved again.
We used separate files so it is more clear and more organized. Each file has its own job, so it is easy to understand and easy to manage. Overall, Pickle helps us save the data, keep the data, and use the data again and again without losing it.

**Error Handling**

In our system, we handled errors by checking inputs and using different exceptions. We did not use only one error, we used different errors for different problems.
We used LoginError for wrong login, ValidationError for wrong input like empty fields or wrong email, and BookingError for booking problems like wrong access, full capacity, or taken time slot.
The system checks things again and again. It checks input, checks access, checks capacity, and checks time slot before booking. So it makes sure everything is correct.
We also used try and except in the GUI. When there is an error, a message shows to the user. So the user knows the problem.
Overall, the system checks errors, shows errors, and helps the user fix errors easily.


-----------------------------------------------------------------------------------------------------------------------------------------------



**Test cases reflection :**

The Smart Campus Facility Booking System effectively handles the essential tasks for both students and administrators. Our test cases show that the user profile section works well, letting students manage their own contact details while keeping their assigned access levels secure. The booking process is also very smooth, the system automatically calculates the total cost and hours whenever a change is made to a reservation. This makes it easy for students to modify their plans without any manual errors.
On the admin side, the dashboard provides a great overview of everything happening on campus. The ability to monitor every single booking and manage user accounts ensures that the system stays organized and secure. The reporting feature is especially helpful because it summarizes daily activity and shows which rooms are being used the most. Seeing that Study Room B is busier than the Event Hall, for example, helps in understanding facility demand. Overall, our tests confirm the system is reliable and user-friendly.

-----------------------------------------------------------------------------------------------------------------------------------------------



**Summary:**

Working on this facility booking system was a great way for us to see how programming actually works in the real world. Throughout the project, our main focus was on using Object-Oriented Programming (OOP) to make the code organized and easy to manage. By creating a main User class and then using inheritance for Students and Administrators, we were able to give different permissions to each person without rewriting the same code. This made the system much more efficient because a student can only book rooms, while an admin has the power to manage everything.
One of the most interesting parts was learning how to keep the data safe using the pickle module. In our earlier projects, everything would disappear when we closed the program but now, the booking history and user accounts are saved into .pkl files. This makes the app feel like a real piece of software that people can actually use day-to-day. We also spent a lot of time on error handling to make sure the program doesn't crash. For example, if a student tries to book a "Premium" room but they only have a "Standard" account, the system stops them and explains why.
The GUI part using Tkinter was also a big learning curve for the group. We had to think about the layout from a student's perspective to make sure the dashboard was simple and not confusing. Seeing the logic from the backend finally show up on a screen with buttons and labels was very satisfying. Overall, this assignment taught us how to connect different parts of Python, like logic, data storage and interface design into one working project. It really helped us understand how to build a complete application from scratch.
