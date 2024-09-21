**Connectivity Context**

The `connectivityContext` is an object that stores relevant data related to the user's booking process. It serves as a container for all the information gathered throughout the booking process, from Quote to Booking.

**Technical POV:**

* The `connectivityContext` is stored in Redis using the session ID as the key.
* Each step of the booking process creates a new object within the existing `connectivityContext`.
* The `connectivityContext` contains a nested structure of objects, with each step adding its own set of data to the existing context.


**Step 1: Quote**

* **User's Perspective:** The user enters their origin, destination, and dates (departure and return).
* **Technical POV:** A new object is created within the `connectivityContext` to store the Quote information.
	+ Key: session ID
	+ Value:
		- Origin
		- Destination
		- Departure date
		- Return date

**Step 2: Prebooking**

* **User's Perspective:** The user reviews and customizes the Quote by selecting options such as flight class, accommodation type, and add-ons like airport transfers.
* **Technical POV:** A new object is created within the existing `connectivityContext` to store the Prebooking information.
	+ Key: session ID
	+ Value:
		- Quote information (from Step 1)
		- Flight class
		- Accommodation type
		- Add-ons (airport transfers, etc.)

**Step 3: Booking**

* **User's Perspective:** The user provides their personal information, enters details for each passenger, selects a payment method, and chooses an insurance option.
* **Technical POV:** A new object is created within the existing `connectivityContext` to store the Booking information.
	+ Key: session ID
	+ Value:
		- Prebooking information (from Step 2)
		- Personal details
		- Passenger details
		- Payment method
		- Insurance option
