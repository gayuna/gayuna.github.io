---
title: A Backend Redesign Exercise - Runday
categories:
  - System Design English
tags:
  - English
---

When studying system design, most resources are written in English and primarily focus on English-speaking services. As a result, Korean developers often face an additional challenge—not only do they need to understand system design concepts, but they also have to familiarize themselves with the services being discussed. This can make the learning process more complex. To address this, this post will explore system design using a well-known Korean service as a base while also drawing connections to similar international services. [See other posts](https://gayuna.github.io/categories/#system-design)

This time, we’ll be looking at a service called “Runday.” Runday was originally developed and launched by HanbitSoft, and is now run by a separate company called Ttam Inc. It’s a running coach app that guides users with step-by-step voice instructions to help them build consistent running habits. The app offers training programs tailored to runners of all levels—from beginners to experienced runners—and includes features that track workout data and support goal setting. In this post, we’ll focus on designing the system behind one of Runday’s core features: choosing a running plan, starting a run, and saving or viewing past records.

#### Workout Flow in Runday

Let’s walk through the typical flow of a workout session in Runday, from start to finish.

![Image](https://github.com/user-attachments/assets/57e3d012-b058-4ac8-9fbf-818f5b11be46)
![Image](https://github.com/user-attachments/assets/558352ed-f34a-48ab-8e01-cfec920a7de1)

First, the user downloads a running course from the app. Then, they choose one of the downloaded courses and tap the “Start Workout” button to begin.

![Image](https://github.com/user-attachments/assets/e6199957-7021-45ea-bc7e-393e5971d88b)
![Image](https://github.com/user-attachments/assets/6afbe5ab-6251-49ce-bf91-87dbbb0ca02c)

Once the workout starts, voice instructions are provided based on the selected course, and the user follows along. During the workout, they can pause and resume at any time.

![Image](https://github.com/user-attachments/assets/ca78186f-f831-4a21-b67e-3fae4a55fc1e)
![Image](https://github.com/user-attachments/assets/bba58f05-9139-4905-bb2a-2e8ba3369e0b)

When the workout ends, the user can review key details such as distance and pace for that session.

![Image](https://github.com/user-attachments/assets/b44a23e7-5619-405d-a975-67bb3cd4a7e1)

The saved workout history can be checked later on the “Workout History” screen.

#### Requirements

Before designing a system, it’s important to clearly define the requirements. This includes identifying the key features the system should support, as well as non-functional requirements like handling increased traffic or concurrency. Based on the workout flow explained earlier, let’s break down the system requirements.

##### Functional Requirements

A workout coaching app like Runday should support the following basic features:
* Users should be able to search, view, and start running courses.
* *When a course is started, coaching instructions should play according to the course. Users should be able to pause or stop the session at any time.
* Users should be able to view their own workout history and also check their friends’ workout summaries.

To keep the focus on the workout flow, this post won’t cover the following:
* Searching for and adding friends
* Community features like creating or joining crews
* Race features
* Sending cheers or encouragement

##### Non-Functional Requirements

* Given the nature of a workout app, availability is likely more important than strict data consistency.
* Once a course has been downloaded, users should be able to continue their workout even if the network connection is temporarily lost.
* The system should handle multiple users starting workouts at the same time without any issues.
* Since the app needs to continuously save the current state during a workout, it’s expected to be more write-heavy than read-heavy.

#### Core Entities

We’ll define the main entities expected to be part of the system. This is an initial outline of what comes to mind, and the structure can be adjusted or expanded later during the actual design process. ID fields for each entity are omitted for now.

* User
  * Represents the user
  * Friend[], Activity[]
* Course
  * A workout course provided by Runday
  * name, description, duration, Step[]
* Step
  * The smallest unit that makes up a Course (e.g. slow walk, light jog, brisk walk)
  * type, duration, music[]
* Activity
  * A record of a user doing a workout based on a specific Course
  * userId, courseId, dateTime, totalTime, startedAt, finishedAt, status, activityRecordId
* Location
  * Real-time location data recorded during a workout
  * activityId, timestamp, lat, lng, altitude
* ActivityRecord
  * Detailed results shown after the workout ends
  * activityId, totalDistance, totalCalories, averagePace, StepRecord[]
* StepRecord
  * Detailed result for each Step in the workout
* Friend
  * Represents a friend, referring to another user
  * userId

#### High level Design

In a system design interview, you’re expected to outline the overall architecture within a limited amount of time. In a typical one-hour interview, once you factor out the introduction, final Q&A, and behaviour questions, you’re usually left with about 30 to 40 minutes for the actual design.

Since it’s not realistic to come up with a perfect, fully optimized solution in that short time, it’s a good idea to focus on defining the key components and APIs during the High-Level Design phase. You can then suggest discussing detailed optimizations later in the Low-Level Design phase.

##### Feature: User Downloads and Starts a Course

When thinking about how users start a course, it feels similar to how playlist features work in music apps. Selecting a course is like picking a playlist, and inside that course are multiple Steps arranged in a specific order. The user follows them one by one.

![image](https://github.com/user-attachments/assets/519e2cb1-a275-475b-b806-c094658c5de8)

The overall setup can be structured with the client on the left and the database on the right. The database stores Course and Step details. The user first checks the list of available Courses, then downloads the full details for a selected Course.

![Image](https://github.com/user-attachments/assets/1ca00bba-2241-4e4a-8615-82c1e368c248)
![Image](https://github.com/user-attachments/assets/e451913e-eed7-422d-b519-dbc707128ff3)

In the actual Runday app, multiple Courses are grouped under a theme and downloaded by theme. But to keep things simple, we’ll assume the user is downloading just one Course at a time. Since the app shows a notice about data usage during this process, it’s likely that background music and other related resources are also downloaded to the device at this point.

![Image](https://github.com/user-attachments/assets/6eb82811-77aa-4d61-908a-207cc44d8cd9)

These audio resources are stored in external storage like S3 and are downloaded along with the Course.

##### Feature: User Starts, Pauses, Completes, or Stops a Course

Once a workout starts, the app needs to handle two main tasks:
1. Guide the user through the Steps in the downloaded Course, in order
2. Record the user’s workout progress

The part that guides the user through each Step is handled on the frontend. Here, we’ll focus on how the backend system is designed.

![Image](https://github.com/user-attachments/assets/aba49512-723e-421c-ae69-daa0a56fb737)

When the workout begins, a request is sent to the Activity Service with the start time and the Course ID. This creates a new Activity entity. If the user pauses, stops, or finishes the workout, the app can use a PATCH /Activity API call to update the status. When stopping or finishing, it’s best to also include the finishedAt time for accurate record keeping.

To make the workout history available later, the system needs to collect location data throughout the session. The client should regularly send the current location and timestamp to the server. If possible, pace and other relevant data should be included in these updates as well.

##### Feature: User Views Workout Summary After Finishing

After ending a workout—or by selecting a specific session from the workout history screen—the user can view their workout details. They can also quickly check their friends’ workout records through the friends list.

![Image](https://github.com/user-attachments/assets/b1ba7e59-4400-4b32-9e8b-8717f01e7903)

To support this, the system needs to store the Activity data along with the real-time Location information collected during the session. Based on that, the system calculates the workout summary and saves it.

When a user wants to view their record later, the app can call a GET API from the Record Service to retrieve the stored data.

#### Expected Issues

There are three major issues that can be anticipated at this point.

##### Network Disconnection

Since Runday workouts usually happen outdoors, it’s common for the network connection to be unstable or drop entirely. In a system where location data is sent every few seconds, missing even a few data points can significantly affect total distance and pace calculations.

A practical solution would be for the app to store the data locally and upload it all at once when the device is back online. In fact, it seems likely that Runday doesn’t stream data to the server in real time. Instead, the app appears to hold all workout data until the user stops or finishes the session, and then sends the full set of data to create the Activity.

##### Handling a High Volume of Users/Data

Because workout apps tend to be used at peak times—like during commutes or lunch breaks—many users may start workouts and upload location data all at once. This can lead to a spike in requests to endpoints like POST /location, putting a heavy load on server resources.

If this pattern continues, it can quickly drain the server’s CPU, memory, and DB connection pool, leading to write delays or timeouts in the database, and potentially resulting in service outages.

And if the service is expected to run long-term, it’s not just about handling spikes in traffic. It’s also important to think about how to manage the growing amount of location data that will accumulate over time.

##### Out-of-Order Location Data

Even without network loss, location data can arrive out of order or get duplicated due to differences in client performance or network delays. For example, the server might receive a location update for 18:30:03 before one for 18:30:00. This kind of ordering issue can cause major inaccuracies in distance tracking.

While availability is the main focus of the system, this type of inconsistency is something users tend to notice the most. If someone puts in the effort to complete a workout but the record is missing or inaccurate, it seriously affects trust in the service. The system needs to correct for these issues as much as possible, and ultimately aim for consistency so users receive accurate workout records.

#### Low Level Design

##### In-memory Buffer + Temporary Local Storage for Location Data

To address network disconnection issues, we can start by temporarily storing location data on the client side using an in-memory buffer. If sending data to the server fails due to internet problems, the buffer will hold the data until the connection is restored. Once the network is back, the client retries sending the data, helping to prevent loss.

Each piece of location data should have a unique UUID. The server should be designed to handle these retries in an idempotent way, so even if the same data is sent more than once, it won’t be duplicated in storage.

##### Handling High Volume of Concurrent Requests: Introducing a Queue-Based Asynchronous Architecture

When a user starts a workout, location data is sent to the server every 3 to 5 seconds. If thousands of users are exercising at the same time, the system may receive several thousand location updates per second. Writing each of these directly to the database could easily create a write bottleneck and lead to system failures.

To avoid this, incoming data should first be pushed into a message queue system such as Kafka, Amazon SQS, or RabbitMQ. The queue buffers the incoming messages, and a separate set of workers (consumers) processes them and writes them to the database in sequence. This approach allows the backend to focus on real-time responsiveness while also supporting horizontal scaling by increasing the number of workers.

Client → API Gateway → Kafka Queue → Location Service → DB

Even if the workers can’t keep up temporarily, the messages will continue to stack in the queue instead of causing a system crash. This setup greatly improves system stability. And if the user base grows and the queue consistently builds up, you can scale out the backend services to handle the load.

##### Sorting by Timestamp to Ensure Data Consistency

Using in-memory buffers or message queues, as mentioned earlier, inevitably makes it harder to maintain strict ordering of location data. However, accurate ordering is important when calculating workout summaries, so each location update should include a timestamp, and the system should sort the data based on that timestamp before using it.

Our current design assumes an RDB as the main data store. While adding an index on timestamp may be fine when the data volume is small, as the number of users grows, the cost of maintaining this index becomes significant. Performance may also suffer when querying location data by activity_id and timestamp, or when aggregating all data for a workout session at the end.

Looking at the nature of the location data—new data being added every few seconds, no updates after insert, write-heavy and append-only—it makes sense to use a different type of database. The data is a good fit for storage that supports fast inserts and is naturally sorted by time. Switching to a more suitable database at this point would likely offer better performance and scalability.

##### Adopting Cassandra

Based on the workload we’ve seen so far for Location data, a distributed NoSQL database is a better fit than a traditional RDB. Cassandra, in particular, supports horizontal scaling and is optimized for high write throughput, making it a strong match for our needs.

Location data is write-heavy and accumulates over time. Since it’s frequently written but rarely updated, Cassandra is well-suited for this kind of use case. It stores data across multiple nodes in a distributed setup, and each node can handle reads and writes independently. As the user base grows, throughput can be scaled linearly just by adding more nodes. Cassandra also buffers writes in memory first and flushes to disk asynchronously when certain conditions are met, which helps it efficiently handle large volumes of write requests.

Cassandra stores and queries data using a partition key. If we design the schema with activity_id + timestamp as the partition and clustering keys, we can store and retrieve location data efficiently in time order. This is ideal for showing a user’s route after a workout or analyzing their session history.

Cassandra works especially well in environments with fixed query patterns. For example, retrieving all location data for a specific activity_id in chronological order can be easily handled by designing the table like this:

```sql
CREATE TABLE Location (
  activity_id UUID,
  timestamp TIMESTAMP,
  location_id UUID,
  lat DOUBLE,
  lng DOUBLE,
  altitude DOUBLE,
  speed DOUBLE,
  PRIMARY KEY (activity_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp ASC);
```

This table uses activity_id as the partition key and timestamp as the clustering key, which ensures that location records within the same activity are automatically sorted by time. This structure is very effective for post-processing tasks like calculating distance or analyzing pace.

That said, Cassandra has limitations when it comes to joins, aggregation, and sorting. So, it performs best in systems with simple and predictable read patterns. As long as you design the data model and queries with these constraints in mind, Cassandra can support a highly scalable and stable system.

##### Hot / Cold Storage Separation

As the app grows and the user base expands, the amount of Activity, Location, and Record data generated daily will grow rapidly. Simple queries that worked fine early on could end up scanning millions—or even tens of millions—of records over time, causing performance issues.

If we think about how users interact with the app, they tend to check recent workout records more often, while sessions from three years ago are rarely viewed. Based on this usage pattern, separating data into a hot table (for recent n months) and a cold table (for older, less frequently accessed data) can help maintain performance and reduce system load.

Even if we’re already using Cassandra, this approach is still beneficial.
* From a disk usage perspective, continuing to insert data over time increases the number of SSTables.
* Cassandra’s write performance is partly due to its background compaction process. But the more large, older data you have, the heavier the compaction cost becomes.

If we separate the tables into hot and cold, we can reduce the load by [minimizing compaction on cold data—or even disabling it entirely if appropriate](https://www.datastax.com/ko/blog/optimizations-around-cold-sstables).

#### Summary
There are many other topics that could be explored at the low-level design stage. For example, if we dove deeper into the cheering feature between friends, we could bring in graph structures. Or, if we considered sending real-time location data to friends, we might look into hotspot issues often seen in social platforms, or even apply design ideas from ride-hailing and delivery apps.

That wraps up the system design for Runday. If I get the chance, I’d like to write more posts like this using other services as system design practice.

#### 참고 자료
[Runday official site](https://www.runday.co.kr/)
[Cassandra Internal Structure](https://nicewoong.github.io/development/2018/02/11/cassandra-internal/)
[[Translated] About Cassandra Writes](https://charsyam.wordpress.com/2012/07/20/%EB%B0%9C-%EB%B2%88%EC%97%AD-cassandra%EC%9D%98-write-%EC%97%90-%EA%B4%80%ED%95%98%EC%97%AC/)
[Cassandra Data Storage – Clustering Key](https://nosqldb.tistory.com/entry/%EC%B9%B4%EC%82%B0%EB%93%9C%EB%9D%BCcassandra-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%A0%80%EC%9E%A5-%EA%B5%AC%EC%A1%B0-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0%EB%A7%81-%ED%82%A4Clustering-Key#google_vignette)