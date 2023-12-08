# Cover Page:

- Subject: Data Science Application Development (DACD)
- Academic Year: 2023-2024
- Degree: Data Science and Engineering (GCID)
- School: School of Computer Engineering (EII)
- University: University of Las Palmas de Gran Canaria (ULPGC)

# Main Functionality

The Weather application provides weather forecasts by integrating with the OpenWeatherMap API. It consists of two main modules: "weather-provider" and "event-store-builder." The first module collects the weather forecast every 6 hours for the next 5 days for the 8 Canary Islands and sends it to an active messaging system or broker (ActiveMQ). The second module, "event-store-builder," subscribes to weather events from the messaging system and stores them in local files, using the following directory structure: "eventstore/prediction.Weather/{ss}/{YYYYMMDD}.events." Here, "YYYYMMDD" is the year-month-day obtained from the event's timestamp, and ".event" is the file extension for storing events associated with a specific day.

# How to Run the Program

The program can be easily executed by following these steps:

1. Download the latest version from the releases of this project. These already include all the necessary dependencies.
2. Unzip the downloaded folders and place them in the desired location.
3. Start the broker. To do this, you will need to download it from this link → [ActiveMQ (apache.org)](https://activemq.apache.org/)
4. Run the 'event-store-builder' module. To do this, from the terminal, use 'cd' to navigate to the directory where the uncompressed folders are located. Use the 'java -jar' command and add the path where you want to store the directory structure as a program argument.
5. Now, do the same with the 'weather-provider' module.
6. In this case, the program argument will be the API key. Obtain your key from this link → [Members (openweathermap.org)](https://openweathermap.org/members)

# Implementation:

## Module "weather-provider".

This module has two packages, control, and model, following the model-view-controller (MVC) pattern. Below, we explain the content of each:

### Control:

#### Class: ActiveMQMessageSender

This class implements the TopicSender interface, defining the method sendMessage(Weather weather) to send weather data to the broker (ActiveMQ). Key functions include:
- sendMessage(Weather weather): Sends a message with weather data to the defined topic.
- InstantSerializer: An inner class to serialize Instant objects to JSON format.

#### Class: OpenWeatherMapSupplier

This class implements the WeatherSupplier interface, defining the method getWeathers(Location location, List<Instant> instants) to obtain weather forecasts from the OpenWeatherMap API. Key functions and features include:
- getWeathers(Location location, List<Instant> instants): Retrieves weather forecasts for a specific location and moments.
- parseJsonDataToWeather(String jsonData, Location location, Instant instant): Converts JSON data into Weather objects.
- buildUrl(Location location): Constructs the URL for querying the OpenWeatherMap API.

#### Class: WeatherController

Controls the periodic retrieval and sending of weather data every 6 hours. Key functions include:
- execute(): Initiates the periodic task of updating and sending weather data.
- calculateForecastTimes(Instant currentTime, int days): Calculates forecast times for the upcoming days.
- loadLocations(): Loads locations for which weather forecasts will be obtained. In the proposed case, the Canary Islands.

#### Class: Main

Initiates the weather forecast application. Configures and runs instances of OpenWeatherMapSupplier, ActiveMQMessageSender, and WeatherController.

### Model:

#### Class: Location

This class represents a geographical location with a name, latitude, and longitude.

#### Class: Weather

This class represents meteorological data events, including information about temperature, humidity, cloudiness, wind speed, probability of rain, among others. Among these, the following are important:
- ts: The date when the forecast is obtained.
- ss: The source producing the data, considered as a constant.
- predictionTime: The day for which the forecast is made.
- location: The location for which the forecast is intended.

This follows the structure of an event, answering the questions: what, where, when, and who.

The following is a class diagram to visually represent these relationships and dependencies between classes:

## Module "event-store-builder":

### Class: FileEventStoreBuilder

This class implements the EventStoreBuilder interface, which defines the save(String message) method to store weather events in local files. Key features and functions:
- save(String weatherJson): Saves weather data in a local file, organized by date and event type.
- getDateString(String weatherJson): Extracts the date of the weather event in a readable format.
- createDirectoryIfNotExists(String ss): Creates directories to store events related to a specific type. This is only used if the file has not been created.

### Class: TopicSubscriber

This class implements the Suscriber interface (defines the start() method) and uses the EventStoreBuilder interface to store weather events locally. Essential functions include:
- start(): Initiates the subscription to weather events.
- processMessage(Message message): Processes incoming messages and saves them using FileEventStoreBuilder.

### Class: Main

Initiates the weather event subscription system. Configures and executes instances of FileEventStoreBuilder and TopicSubscriber.

Similarly to the previous module, the following is the class diagram for event-store-builder:

## Design and Principles

The application adheres to SOLID principles to ensure a robust and maintainable design. Below are specific SOLID principles applied in the implementation:

### Single Responsibility Principle (SRP):

Ensures that each class has a single responsibility. For example, the FileEventStoreBuilder class has the exclusive responsibility of saving weather events to local files. This way, if the storage process needs modification, only this class will be affected.

### Open/Closed Principle (OCP):

Indicates that existing code should only be modified to fix errors and not to add new functionalities. For instance, the weather provider interface allows introducing new information providers without modifying the WeatherController logic.

### Liskov Substitution Principle (LSP):

Subclasses should be substitutable for their base classes without affecting functionality. For example, the OpenWeatherMapSupplier class is entirely substitutable for its base class WeatherSupplier, ensuring that any part of the system using WeatherSupplier works correctly with OpenWeatherMapSupplier.

### Interface Segregation Principle (ISP):

Interfaces should be designed without unnecessary methods, keeping them clean. Large interfaces should be divided into smaller ones. For example, the WeatherSupplier interface offers only the necessary functionality to obtain weather forecasts.

### Dependency Inversion Principle (DIP):

Dependencies are separated to avoid depending on low-level modules. Instead, both should depend on abstractions. For example, the WeatherController class depends on the WeatherSupplier and WeatherStore interfaces rather than their implementations.

### Observer Pattern:

The Observer pattern is used to notify subscribers, in this case, the "event-store-builder" module, about new weather events. This decouples event generation and storage.

### Exception Handling:

A consistent exception-handling system is implemented. In the "event-store-builder" module, custom exceptions like WeatherReceiverException are used to encapsulate specific errors for proper handling. No interface throws exceptions that are not custom, especially Runtime Exceptions.

### Logger Usage:

The use of a logger throughout the code improves visibility and exception handling. Exceptions are appropriately logged, aiding debugging and system monitoring in production.

### Data Lake:

The "event-store-builder" module stores weather events in a local file format organized as a "Data Lake." This approach allows for easy expansion and long-term data analysis.
