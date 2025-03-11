---
title: A Backend Redesign Exercise - Interpark Ticket
categories:
  - System Design
tags:
  - English
---

When studying system design, most resources are written in English and primarily focus on English-speaking services. As a result, Korean developers often face an additional challenge—not only do they need to understand system design concepts, but they also have to familiarize themselves with the services being discussed. This can make the learning process more complex. To address this, this post will explore system design using a well-known Korean service as a base while also drawing connections to similar international services.

The first service we’ll examine is Interpark Ticket. Currently operated by Yanolja after its acquisition, this ticket purchasing platform offers a wide range of tickets for events, from concerts to exhibitions. Similar domestic services include Melon Ticket and Yes24 Ticket, while Ticketmaster is a well-known counterpart frequently mentioned in system design interviews in English-speaking regions. Additionally, movie ticketing apps like CGV and Megabox follow a comparable booking process.

In this post, we will walk through the ticket purchasing process for events with assigned seating, such as concerts and musicals. The content has been adapted from Hello Interview’s System Design Interview: Design Ticketmaster video to align with the Korean service landscape. The goal is to help readers prepare for a system design interview lasting approximately 40 minutes.

We will first design the basic service under the assumption that there is no competition for seats. Then, we will expand the system by incorporating features needed to handle high traffic situations.

#### Following the Basic Flow of a Ticket Booking System

First, let’s go through the typical ticket booking process.
![image](https://github.com/user-attachments/assets/33d35cfe-ca1e-4c59-9577-7b881bcc810a) ![image](https://github.com/user-attachments/assets/39555965-a039-4fd7-826d-64be1ad7640d) 
![image](https://github.com/user-attachments/assets/382cc8be-bb68-4642-bbd9-e881ae0f08fb)

The user starts by accessing the app and entering the name of the concert or artist in the search bar. The search results page displays a list of performances that match the keyword, allowing the user to select their desired event and proceed to the booking screen.

![image](https://github.com/user-attachments/assets/927f56d1-244d-4315-8651-2561b55e1667)

Once the user clicks on the booking button, they can choose the preferred date and time slot for the event.

![image](https://github.com/user-attachments/assets/5f3af4e2-af89-4f0b-8ae3-85c023cde352)
![image](https://github.com/user-attachments/assets/71706347-67b0-424c-a84f-017dc7ebc59a)

At this stage, a seating chart for the venue appears, showing available seats. The user selects a seat, specifies the delivery method, and proceeds to payment.

![image](https://github.com/user-attachments/assets/0655cf56-9f3e-4a0d-9038-aa80fc28a53d)

Once the transaction is complete, the ticket is stored in the user’s “Booking Confirmation” section, where they can review their reservation.

#### Requirements

Before designing the system, it’s important to clearly define the requirements. This includes outlining the core functionalities and considering non-functional requirements such as handling traffic spikes and concurrency issues. Based on the ticket booking flow described earlier, we will analyze the system requirements.

##### Functional Requirements
The ticket booking system should provide the following core functionalities:
* Users can search for events.
* Users can view detailed information about a specific events.
* Users can select a date and seat to purchase tickets.

However, to keep the focus on the core booking flow, the following features will not be covered in this post:
* Events without assigned seating (e.g., exhibitions, festivals)
* User registration
* Adding new performances to the booking service
* Setting different prices based on performances or seat locations
* Pre-sales
* User reviews
* Discount services
* One-click payments

##### Non-functional Requirements

In addition to functional requirements, system design must also account for performance and stability considerations.
* Ticket sales must maintain consistency.
  * The same performance, date, and seat should never be sold to multiple users simultaneously.
  * Therefore, consistency is the most important factor in ticket sales.
* In contrast, performance searches and detailed information retrieval must maintain high availability.
  * Slight delays in updating search results or event details are not critical.
  * As a result, availability takes priority for these functions.
* The system must handle high traffic for popular events.
  * Some highly sought-after performances may have hundreds of thousands of users trying to book tickets at the same time.
  * To address this, the system should support scalability through horizontal scaling and include traffic management techniques such as queuing systems, caching, and rate limiting.

#### Core Entities

The core entities expected in the system are defined below. These are initial definitions, and additional entities may be introduced as needed during the actual design process. Entity IDs are omitted for now.

* User
  * Represents a user making a reservation
  * Attributes: Bookings[] (list of bookings made by the user)
* Event
  * Represents a performance or show
  * Attributes: Title, PerformanceDateTime, Venue, Performer, Description, Tickets[] (list of tickets available for the event)
* Ticket
  * Contains ticket information
  * Attributes: EventId, Status
* Venue
  * Represents the venue where events take place
  * Attributes: Events[] (list of events held at this venue), Seatmap
* Performer: Represents an artist or performer for an event
* Booking
  * Stores reservation details
  * Attributes: UserId, EventId, TicketId, Status

#### High level Design

In a system design interview, the goal is to design the overall architecture within a limited time. Since creating a perfectly optimized design in this short time is unrealistic, it’s best to focus on defining the key components and APIs at the high-level design stage and agree with the interviewer to refine details in the low-level design phase.

##### Designing the Search Functionality

et’s start with the search function. As seen in the screenshots, users enter a performance name into the search bar. (Although not shown here, users can also filter by categories—such as musicals—to view a list of performances within that category.) The search results page allows filtering by genre, date, location, and discount options, and displays a list of matching performances. Each item in the list includes details like performance name, venue, date, rating, and status (e.g., tickets available). We can extend our Event entity to include Status and Rating.

![image](https://github.com/user-attachments/assets/94421c0d-5e7b-43a5-96ff-5f8fd757c3da)

On the left, we have the Client, and on the far right, a SQL database is assumed. Other database options can be discussed later with the interviewer. The system follows a microservices architecture, where the client request first goes through an API Gateway, then to the Search Service, which queries the database and returns results.

![image](https://github.com/user-attachments/assets/a1554c77-1b7a-4fce-803c-bbe454599497)

To visualize the API calls and queries, a GET /search API seems appropriate. This API takes multiple parameters and returns a list of matching events.

##### Retrieving Detailed Event Information

Whether accessed via search results or a banner on the main page, users should be able to navigate to a detailed event page. This page contains more information than the search results, including a detailed description, organizer-provided details, and user reviews.

![image](https://github.com/user-attachments/assets/293e9d29-d148-4b2b-9621-164d26c0735d)

For this, an Event Service is introduced. User requests are routed to this service, which queries the database and returns the required details. The most logical approach is to pass the event ID as a path parameter from the search response.

##### Viewing and Booking Seats

The seat selection and ticket booking process is one of the most important functions in the ticketing service. Based on the screens, after clicking "Book Now," the user selects a date for the event. If there are multiple showtimes on the same day, they also choose a specific time slot.

Next, a seating map is displayed, highlighting available seats. Once the user selects a seat and proceeds, they reach the booking confirmation and payment page. After completing payment, they can review their booking in their account.

Initially, I considered placing the API for date/time/seat selection in the Event Service, but after considering the queueing system, it makes more sense to place this logic in the Booking Service.

![image](https://github.com/user-attachments/assets/39852711-2c23-4915-8a16-a12dd0a813e6)

First, a GET API is created to retrieve the seating chart and available seats for an event. This API is called to display the venue’s seat map and indicate which seats are currently available for booking.

Next, an API is designed to allow users to proceed after selecting their desired ticket. This is handled by the reserve API, which, when called, temporarily holds the selected seat. The Booking Service creates a Booking entity containing eventId, showtime, and the selected ticket details. The response includes user preferences such as saved shipping and payment details.

Finally, the confirm API is implemented to finalize the booking. Users can modify their Booking details before submitting a confirmation request. The Booking Service processes payment using an external payment module. If the payment is successful, the Booking and Ticket Status are updated to completed.

At this point, the functional requirements seem largely covered. However, there are still non-functional issues to address.

#### Expected Issues

When designing a ticket booking system, two major issues are likely to arise.

##### Concurrent Booking Conflicts

Consider a scenario where users A and B attempt to book the same seat for a performance at the same time. At the moment both users request the seat map, Row 3, Seat 15 appears to be available. Since their requests occur almost simultaneously, both may attempt to reserve the seat.

In this case, the system must ensure that the first user to complete the reservation gets the seat, while the other fails immediately. The key point here is that the failure must happen instantly—if the system only informs the user at the final payment step that the seat is no longer available, it would result in a poor user experience.

![Image](https://github.com/user-attachments/assets/f1af9c33-1053-4b8f-867c-9ecd415ff90c)

To handle this, services like Interpark Ticket prevent users from proceeding past the seat selection step if the seat has already been taken. Instead, the second user receives a “Seat already reserved” notification and is prompted to refresh the seat map before continuing.

This approach prevents unnecessary steps in the purchasing process and improves the overall user experience.

##### High Traffic Issues

When ticket sales open for a highly anticipated event, a massive number of users may attempt to access the system at the same time. This can put the server under extreme load. Even if the system is scaled out in anticipation of 15 times the normal traffic, an actual surge of 50 times the traffic could still lead to system failures.

An even bigger challenge is that, even if the ticketing system itself is scaled out, external services such as payment processors may not be able to scale in the same way due to their own limitations.

Another key consideration is that even during peak traffic periods, the rest of the site must continue functioning properly.
For example, if Coldplay tickets go on sale at 8 PM on a Thursday, users trying to check their theatre reservations at the same time must not experience disruptions. Likewise, on-site customers picking up their tickets should not face issues due to overloaded servers.

To address these concerns, services like Interpark Ticket use dedicated landing pages for high-traffic events instead of directing all users to the main site. This page allows users to either go directly to the high-traffic event’s booking page or proceed to the main site for other ticketing needs.

![Image](https://github.com/user-attachments/assets/6c9368e7-bdbc-4e75-91e2-efb1ab168de7)

Additionally, many platforms implement queueing systems when a large number of users attempt to book tickets at the same time. Users are assigned a queue position and must wait their turn. Only a limited number of users are allowed into the seat selection page at any given time, preventing system overload.

This approach ensures fairness while also preventing excessive strain on the infrastructure.

#### Low level Design

To Improve the previously discussed issues, let’s refine the initial design further.

##### Preventing Simultaneous Seat Reservations

To prevent two users from booking the same seat at the same time, one of the first approaches is to introduce an additional status field in the Ticket entity.

By default, a ticket can have the following statuses:
* AVAILABLE – The seat is available for booking.
* BOOKED – The seat has been successfully reserved and paid for.
* ON HOLD (or RESERVED) – The seat is temporarily reserved by a user who is in the process of completing the booking.

To enhance this, an expiration time (expired_at) field is introduced. When a reservation request is made, the ticket’s status changes to RESERVED, and the system records the reservation time. The user then has a limited period (e.g., 5 minutes) to complete the purchase. Any latecomers attempting to book the same ticket will fail, as it is no longer in the AVAILABLE state. If the initial user fails to complete the booking before the expired_at time, the ticket automatically becomes available again.

At the seat selection stage, the system must retrieve not only AVAILABLE seats but also RESERVED seats where the expired_at timestamp has already passed.

While this approach works in normal conditions, it can lead to performance bottlenecks when handling a large number of concurrent bookings. If thousands of users attempt to reserve seats at the same time, the database will experience constant read/write operations, increasing the risk of contention. In high-demand events, multiple users may attempt to reserve the same seat almost simultaneously, causing a surge in database transactions.

Thus, relying solely on the status-based method may not be enough for high-traffic scenarios. To mitigate this, another approach is to implement a distributed lock mechanism. This ensures that when one user begins the reservation process for a particular seat, others must wait until the transaction is completed or expired. This method significantly reduces conflicts and prevents excessive database load.

[If we introduce Redis Lock](https://varunkruthiventi.medium.com/distributed-locking-with-redis-af4e7d80b503), the system will check both whether the ticket is in an AVAILABLE state and whether a lock already exists.
Since Redis is single-threaded, it maintains data consistency even in a scaled-out environment. This ensures that no two users can acquire a lock on the same seat at the same time.
If a lock already exists, it means another user has secured the seat, so the current user is redirected to the previous screen. If no lock exists, a new lock is created with a TTL (Time-to-Live), allowing the user exclusive access to proceed with the reservation for a limited time.
If the booking is completed within the allocated time, the ticket status changes to BOOKED, preventing others from reserving it. However, if the reservation is not finalized before the lock expires, the lock is automatically removed, making the seat available again for other users.

![Image](https://github.com/user-attachments/assets/263e5ae4-9d5b-44f4-8577-335f589b5ab3)

Of course, it is unclear which specific approach Interpark Ticket uses. However, based on the observed behaviour where unsold seats become available again a few minutes after ticket sales begin—often referred to as “ghost tickets”—it seems likely that they are using a status-based approach. Additionally, since these ghost tickets appear to be released sequentially from front-row seats, it suggests that instead of dynamically checking expired_at, there might be a separate cron job running in the background. This job could periodically scan for tickets with expired reservations and update their status back to AVAILABLE at set intervals.

It would be worthwhile to discuss the pros and cons of each method with the interviewer.

##### Using Cache to Handle High Volumes of Event Information Requests

When ticket sales begin for a major concert or popular event, a large number of users request the same event information simultaneously. If the system queries the database for each request, it can lead to unnecessary load and performance degradation. To prevent this, a caching strategy is a natural solution.

Event details are relatively static data—once finalized, they rarely change. Information such as venue, performers, and event dates has a low likelihood of modification, allowing for a longer TTL (Time-to-Live) setting in the cache without concerns. If an update does occur, an automatic cache invalidation mechanism can be implemented to ensure the latest data is served while maintaining optimal performance.

One noticeable aspect of Interpark Ticket’s event detail pages is that they are heavily image-based rather than text-heavy.

![Image](https://github.com/user-attachments/assets/be585f07-4109-4181-a64a-18118bdced42)
![Image](https://github.com/user-attachments/assets/6e35c983-3138-4aae-8b54-58433db58f8e)
![Image](https://github.com/user-attachments/assets/fc2c98de-8221-49aa-be0e-470de05d5dbf)

With this kind of structure, it is more efficient to distribute static content like images through a CDN (Content Delivery Network) rather than serving them directly from the server.

Benefits of Using a CDN:
* Faster image delivery – Images are cached on distributed edge servers, reducing load times.
* Lower server load – Requests are handled by nearby CDN nodes instead of reaching the origin server, minimizing unnecessary traffic.
* Better scalability – During high-traffic periods, such as ticket sales, the CDN absorbs most of the requests, ensuring stable system performance.

By storing event details in cache instead of querying the database and using a CDN for static resources, the system can significantly reduce database and server load while providing faster response times.

This design is not only beneficial for ticketing systems but is also a fundamental architecture principle for any web service that experiences sudden spikes in visitor traffic.

##### Issuing Queue Numbers and Isolating Events

In high-traffic situations, allowing all users to access the booking screen at the same time is not feasible. As previously mentioned, a waiting room system can be introduced to control traffic flow by allowing only a manageable number of users to proceed to the booking screen in sequence.

Waiting room systems are not exclusive to ticketing services—they are widely used in various industries, including:
* Delivery services – During coupon distributions or promotional events.
* E-commerce – For high-demand product launches, such as flagship smartphones.
* Flight ticket sales – When discounted airline tickets are released.

Managing a queue in high-traffic environments is a well-established architectural pattern. Cloud providers such as Cloudflare and AWS offer solutions for this under the name Virtual Waiting Room, helping businesses handle extreme traffic surges effectively.
* [Cloudflare Waiting Room](https://www.cloudflare.com/ko-kr/application-services/products/waiting-room/)
* [AWS Virtual Waiting Room Architecture](https://aws.amazon.com/ko/blogs/tech/getting-started-virtual-waiting-room-on-aws/)

When implementing a queue system, Redis is commonly used, as seen in the cases of [Baemin](https://www.youtube.com/watch?v=MTSn93rNPPE) and [Wonderwall](https://tech.wonderwall.kr/articles/vwr/). Baemin utilizes a Sorted Set, while Wonderwall takes an approach using atomic increment as a counter. Despite these differences, both systems assign queue numbers in the order users arrive and allow users to enter in that sequence—either by moving them to the waiting area or issuing a token. For more details, it is recommended to check the implementation strategies of these companies.

Using a queue-based approach helps ensure that the backend only receives traffic at a manageable rate. Additionally, the state of the virtual queue can be monitored in real time, allowing the system to dynamically scale out as needed based on the traffic load.

Once a user exits the queue, it may be beneficial to handle the request using a dedicated instance rather than relying on a shared Booking Service used for multiple events. Running a separate instance for a high-demand event allows the system to precisely control the number of users entering from the queue and prevents interference with other event bookings.

For Interpark Ticket, when large traffic spikes are expected, the main page changes to let users explicitly select which event they want to book before proceeding. From this point, the system is likely handling everything based on eventId. On the other hand, Baemin takes a different approach by preloading a list of store IDs for an event (e.g., a chicken restaurant list for a chicken promotion) and placing an order router in front. If an order is placed at a store included in the list, it is routed accordingly.

#### Conclusion

There are many other topics that could be explored in low-level design. For instance, if the service is operated for an extended period, the number of event records will grow significantly, leading to longer search times. One possible optimization strategy—assuming we are using an RDB like MySQL (as mentioned earlier)—is to leverage Full-Text Indexing to improve search performance.

In a system design interview, it is often beneficial to steer the discussion toward technologies you have hands-on experience with. This not only demonstrates practical knowledge but also makes it easier to explain trade-offs and decisions effectively.

This concludes the Interpark Ticket system design discussion. If the opportunity arises, I plan to explore the system design of other services in future posts!
