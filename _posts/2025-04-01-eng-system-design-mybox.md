---
title: A Backend Redesign Exercise - MY BOX
categories:
  - System Design English
tags:
  - English
---

When studying system design, most resources are written in English and primarily focus on English-speaking services. As a result, Korean developers often face an additional challenge—not only do they need to understand system design concepts, but they also have to familiarize themselves with the services being discussed. This can make the learning process more complex. To address this, this post will explore system design using a well-known Korean service as a base while also drawing connections to similar international services. [See other posts](https://gayuna.github.io/categories/#system-design)

The service we’ll be looking at this time is "MY BOX". MY BOX is a cloud storage service provided by Naver, offering users a space to securely store and manage various types of files such as photos, videos, and documents. It can be easily accessed through both PC and mobile apps, and includes several user-friendly features such as automatic backup, folder sharing, and media preview. In this article, we’ll focus on one of the most central features of MY BOX: uploading, storing, browsing, and downloading files.

#### MY BOX File Upload / Download – Walking Through the Basics

Let’s walk through the basic process of uploading and downloading files.

![Image](https://github.com/user-attachments/assets/bb53d071-5395-4134-b2a9-b4ec567b3c2b)

First, a file to be stored is uploaded. When you go to the photo upload menu, you can select files stored locally on your device. 

![Image](https://github.com/user-attachments/assets/5e036c0c-d14f-495d-9954-5ae475e854db)

After choosing the file to upload, the process begins.

![Image](https://github.com/user-attachments/assets/14260f66-a013-4164-8e27-2df8bcb78d45)

Once the upload starts, there’s a brief “preparing to upload” phase, and then the file is uploaded. 

![Image](https://github.com/user-attachments/assets/8a9631b4-c048-48a0-ac0d-c5a08ba44655)

Once the upload is complete, the time of upload is shown in the upload history.

![Image](https://github.com/user-attachments/assets/ea7dd43f-5302-447e-8ed7-d7389ab2f586)

For downloading, it works the other way around. You start by selecting the file you want to download from those saved in MY BOX. 

![Image](https://github.com/user-attachments/assets/32572964-65ea-43d8-bb1b-e4d5855778c1)

When the download begins, the file enters a brief waiting state, and after a short time, the download completes. 

![Image](https://github.com/user-attachments/assets/d9b698f8-3189-4bfd-8dac-7aa6eb572934)

The time of completion is also displayed.

#### Requirements

To design a system, the first step is to clearly define the requirements. It’s important to outline the main features the system should support, along with non-functional requirements such as handling traffic spikes and concurrent access. Based on the upload/download flow discussed earlier, let’s go over the requirements for this system.

##### Functional Requirements

MY BOX, as a cloud storage service, should provide the following basic features:
* Users can upload files.
* Users can download files.
* The service should be accessible from both PC and mobile devices.

To keep the focus on the basic upload/download flow, the following features are not covered in this post:
* Managing blob storage
* File sharing
* Online streaming
* Support for different file sizes depending on purchased plans
* Features like facial recognition and categorization

##### Non-Functional Requirements

* In terms of viewing files, availability is considered more important than data consistency. This is because the service does not rely on showing an uploaded file immediately after it’s been uploaded. The goal is to prioritise overall system availability and aim for eventual consistency.
* However, data integrity matters. Files must not be corrupted or altered.
* Uploading and downloading should not be too slow. (Latency)
* Large files such as videos (10GB to 50GB) should be supported. To handle this, users should be able to pause and resume uploads or downloads.

#### Core Entities

We’ll define the main entities expected to be part of the system. This is an initial outline of what comes to mind, and the structure can be adjusted or expanded later during the actual design process. ID fields for each entity are omitted for now.

* User
  * Represents the user
* File Metadata
  * FileName, dateUploaded, FileSize, FileUrl, UserId
* File
  * The file itself

Let me know when you’d like to continue.

#### High level Design

In a system design interview, you’re expected to outline the overall architecture within a limited amount of time. In a typical one-hour interview, once you factor out the introduction, final Q&A, and behaviour questions, you’re usually left with about 30 to 40 minutes for the actual design.

Since it’s not realistic to come up with a perfect, fully optimized solution in that short time, it’s a good idea to focus on defining the key components and APIs during the High-Level Design phase. You can then suggest discussing detailed optimizations later in the Low-Level Design phase.

##### Feature: File Upload by the User

Let’s start by thinking about how a user uploads a file. For now, we’ll focus on small files—those under 100MB.

![Image](https://github.com/user-attachments/assets/688f8a7f-3df1-4206-a035-c48b1fb4d07b)

Whether on PC or mobile, when a user selects a file to upload, the client calls the `POST /files` API. The file and its metadata are sent together in `multipart/form-data` format. The File Service parses the request, stores the file in an object storage like S3, and saves the file’s URL along with its metadata in the File Metadata DB.

After uploading a file in MY BOX, users can see a list of uploaded files, each with a thumbnail. It makes sense to generate the thumbnail during the upload process. However, since this is a design detail and not critical to the main functionality, it’s better not to create thumbnails synchronously. Instead, we’ll use a separate job queue and a dedicated Thumbnail Service.

![Image](https://github.com/user-attachments/assets/d1b178bf-6dd0-4381-9800-705cb2e95abf)

While uploading, the File Service sends a request to a thumbnail generation queue. The Thumbnail Service processes the request from the queue, creates the thumbnail, stores it in S3, and then saves the thumbnail’s S3 URL in the metadata DB. We’ll need to add a thumbnailUrl field to our file metadata entity.

##### Feature: File Download by the User

Now let’s look at the download flow. To download a file, the user first needs to see a list of their files. By calling the `GET /files` API, they can view the uploaded files. It makes sense to support pagination, such as `GET /files?page=1&pageSize=20`. This GET API returns basic metadata for each file to display in the list—such as the file ID, thumbnail, upload time, and size.

![Image](https://github.com/user-attachments/assets/6a230996-4d93-40c1-a38b-2de843ab5798)

When the user selects a file from the list to download, the `GET /files/{fileId}` API is called. The File Service uses the fileId to retrieve metadata, checks the file’s location in object storage, and reads the actual file to send back to the client as binary data. Files are sent using a streaming method.

![Imag](https://github.com/user-attachments/assets/00da55a7-1c15-4379-92bd-17125fa8e8b2)


Streaming means the file isn’t loaded into memory all at once. Instead, the server reads data in chunks from object storage and sends it to the client at the same time. This approach helps reduce memory use and ensures stable file transfers. Most backend frameworks support streaming out of the box. For uploads, streaming is usually handled automatically. Downloads can be implemented with relatively simple code.

In the commonly used Java and Spring combination in Korea, Java’s InputStream and OutputStream classes can be used, along with Spring utilities like StreamUtils. For example, StreamUtils.copy(in, out); connects the stream reading data from S3 with the stream sending it to the client. Node.js also provides its own Stream module, which supports similar streaming behaviour.

#### Expected Issues

##### Heavy Load and Network Interruptions During Large File Uploads/Downloads

MY BOX allows users to store files up to 10GB–50GB depending on their plan. When uploading such large files through the server, the server has to handle both 10GB of incoming and 10GB of outgoing traffic simultaneously. If multiple users attempt uploads at the same time, the load can affect not only network bandwidth but also system-wide resources such as memory, disk buffers, and garbage collection.

Users may also experience network interruptions or close their browser during large uploads or downloads. If the transfer has to start from scratch in those cases, the experience becomes frustrating. This issue is more common on mobile or slower network environments.

##### Storage Concerns for Large Files

With frequent uploads and long-term storage of large files, several challenges can arise.

From a storage perspective, ongoing accumulation of large files can lead to limited space, higher costs, and slower read/write performance. If rarely used files remain in expensive general-purpose storage, this can result in significant long-term costs.

Also, storing a file is not the end of the process—additional data like thumbnails, metadata, and access logs must also be managed. This increases the risk of inconsistency. For example, a file may be uploaded successfully, but its metadata may not be saved, showing incomplete information to the user.

In such cases, the system must ensure data integrity. A file must not be corrupted or partially saved. On the other hand, trying to keep all data completely in sync at all times may introduce delays and reduce overall system availability.

To balance integrity and availability, we plan to adopt an eventual consistency model, allowing slight delays in processing and reflecting related data.

##### User-Perceived Speed and Response Time

Slow uploads and downloads can cause users to leave the service. For large files, strategies such as splitting the transfer into smaller parts and handling them in parallel can help reduce latency.

As the time to handle a single request increases, server load also grows, and users may encounter usability issues. To improve the experience, it’s important to provide progress indicators, step-by-step responses, and streaming-based delivery where possible to keep users informed throughout the transfer.

#### Low-level Design

Handling large file uploads and downloads reliably is a common challenge faced by many services. There are various solutions available, including open-source options. One example is tus, an open-source protocol for resumable uploads. It’s designed so that if the network connection is interrupted during an upload, the client can resume from where it left off. Since it’s a simple HTTP-based protocol, it can be integrated into many environments with ease, and there are implementations available for multiple languages and platforms.

That said, simply mentioning the use of an open-source tool wouldn’t make for a very informative post, so instead of using tus, this article will explore a design approach based on AWS S3’s Multipart Upload and HTTP Range requests. We’ll walk through how to build a client-driven upload and download flow using those tools directly.

##### Supporting Large File Uploads

To upload files faster without overloading the server’s bandwidth, it’s best to let the client upload directly to storage. Instead of sending the file through the backend, the initial API call is only used to prepare for the upload and store metadata. The actual upload is handled directly by the client. AWS S3 provides a feature called Multipart Upload for this exact use case.

[Multipart Upload](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/mpuoverview.html) is a method offered by AWS S3 for uploading large files. It breaks a single large file into smaller parts that are uploaded separately and then combined into a single object. It’s similar to sending a 10-volume encyclopedia by courier—if it can’t fit in one box, you ship it in 10 separate boxes and assemble the set once all the boxes arrive. AWS recommends using multipart upload for objects over 100MB, which is why we used 100MB as the threshold for “small files” in the high-level design.

Here are the main terms used in Multipart Upload:

| Concept         | Description                                                               |
|--------------|----------------------------------------------------------------------|
| Part     | A chunk of the full file. Each part must be at least 5MB, and up to 10,000 parts are allowed. |
| PartNumber | The number assigned to each part—1 for the first part, 2 for the second, and so on. |
| UploadId | A unique ID used to identify the upload session. It must be used until all parts are uploaded. |
| ETag     | A unique hash value returned by AWS when each part is uploaded. This is required for final assembly.|
| Complete | The step where you tell AWS “All parts have been uploaded; please combine them into one file.” |

During the upload process, we need to keep track of the UploadId, PartNumbers, and other related information. While the client could manage this on its own, considering potential issues like network interruptions and large data sizes, it’s better to let the backend handle it. This way, the client is only responsible for sending the actual file data, while the server takes care of managing state and overall flow.

Given the nature of this data, the number of parts and other details will vary significantly depending on the file size. So instead of adding all the related fields to the existing metadata RDB, it’s more appropriate to store this information in a separate NoSQL database. This would allow for fast read/write performance, automatic expiration via TTL (important because AWS charges for incomplete uploads), and quick access based on keys, without the need for complex queries.

Since we’re already using AWS S3, we’ll use DynamoDB. It’s AWS’s NoSQL offering and fits our needs perfectly. We’ll add a key field to the existing metadata DB to reference entries in DynamoDB. Also, because uploads might remain in progress for a while, it makes sense to add a ‘status’ field to the metadata.

![image](https://techblog.woowahan.com/wp-content/uploads/2023/08/Spring-Boot%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5-S3%E1%84%8B%E1%85%A6-%E1%84%91%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%AF%E1%84%8B%E1%85%B3%E1%86%AF-%E1%84%8B%E1%85%A5%E1%86%B8%E1%84%85%E1%85%A9%E1%84%83%E1%85%B3%E1%84%92%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB-%E1%84%89%E1%85%A6-%E1%84%80%E1%85%A1%E1%84%8C%E1%85%B5-%E1%84%87%E1%85%A1%E1%86%BC%E1%84%87%E1%85%A5%E1%86%B8-%E1%84%8B%E1%85%B5%E1%84%86%E1%85%B5%E1%84%8C%E1%85%B5-9.jpg)

The client calls the `POST /files/init-multipart` API to prepare for uploading a large file. It sends information such as the file name, size, and content type to the server. Based on this information, the server initiates a Multipart Upload in S3 and stores the Upload ID along with the file’s metadata so it can manage the upload state. The server then returns a list of pre-signed URLs along with the UploadId, which the client can use to upload each part directly to S3. Since parts don’t have to be uploaded in order and can be uploaded in parallel, the upload can be completed much faster. This process likely takes place during the “Preparing to upload” stage that we saw earlier.

The client uploads each part to the corresponding pre-signed URL. Once a part is uploaded, S3 returns an ETag. The client must notify the server that the part has been uploaded by calling an API like `POST /files/{fileId}/part-complete`, sending the ETag in the request body. The server stores the ETag and updates the upload status. If all parts for that file are marked as uploaded, the server sends a final request to S3 to indicate that the upload is complete. At that point, S3 assembles the file using the ETags.

![Image](https://github.com/user-attachments/assets/2c478f48-a93a-4cf4-9be7-33c897b99de7)

Using AWS Multipart Upload helps resolve bandwidth issues since files are uploaded directly from the client to S3 without passing through the server. Because each part is uploaded independently, uploads can happen in parallel and in any order. This also makes it possible to pause and resume uploads—if the network connection is interrupted, only the parts that failed need to be uploaded again.

For a more detailed explanation of the upload process, you can refer to the blog post “[Three ways to upload files to S3 using Spring Boot](https://techblog.woowahan.com/11392/)” from Woowa Tech Blog (in Korean).

As mentioned earlier, AWS charges for files even if the upload wasn’t completed. So if an upload is left hanging for too long, the file should be cleaned up from S3. [S3 supports lifecycle policies that automatically abort incomplete multipart uploads after a set period of time from the initial initiation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpu-abort-incomplete-mpu-lifecycle-config.html). If we set this period to match the TTL configured in DynamoDB, then both the upload metadata on our server and the incomplete file in S3 will be cleared at the same time.

In addition, we should regularly check for files that haven’t been completed within this time frame and update their status to “aborted” in the metadata DB.

##### Supporting Large File Downloads

Large file downloads can be handled in a way similar to uploads by allowing the client to download directly from S3 using pre-signed URLs. The client can use HTTP Range requests to resume downloads from where they left off if interrupted.

![Image](https://github.com/user-attachments/assets/8af587d8-78ab-4e84-b348-88e82aa6f3ef)

HTTP Range requests allow the client to specify a particular byte range to download. For example, sending a header such as `Range bytes=1000000-2000000` instructs S3 to return only that portion of the data. This approach enables resuming downloads without starting over, and it also allows for parallel downloads.

The client can split the file into multiple chunks and send parallel requests with a combination of pre-signed URLs and Range headers for each chunk. This not only speeds up the download but also improves the experience, since only the failed chunk needs to be re-downloaded if an interruption occurs. This process can be integrated during the “waiting” phase observed earlier.

State management for downloads should be handled by the client—using memory or local storage—without requiring backend intervention. However, if a download is paused for an extended period, the pre-signed URL may expire. In such cases, the client should call the `GET /files/{fileId}/download-url` API before resuming to obtain a new pre-signed URL. This ensures that the client can send HTTP Range requests without authentication errors, enabling a reliable download resume process. For very large files, since the pre-signed URL might expire even during uninterrupted downloads, the API response could include an expiredAt field. If the download is still in progress as this time approaches, the client can request a new URL to maintain a continuous download.

##### Introducing Cold Storage to Save Costs

In systems where large files are stored for long periods, storage costs can become a significant concern. When users frequently upload large files but only access a small portion of them regularly, most files end up in a “stored but rarely accessed” state. For these files, it makes sense to move them to a lower-cost option, even if that means slower access times.

Since we’re already using AWS S3 for file storage and DynamoDB for state management, the most natural approach is to adopt one of AWS’s cold storage options, such as S3 Intelligent-Tiering or S3 Glacier. [AWS storage class documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)

[S3 Intelligent-Tiering](https://docs.aws.amazon.com/AmazonS3/latest/userguide/intelligent-tiering.html) automatically monitors access patterns and moves frequently accessed files to the Standard tier, while less frequently accessed files are placed in the Infrequent Access tier. The benefit here is that no restoration time is needed—files remain instantly accessible. This tier is useful when it’s hard to predict access patterns.

[S3 Glacier](https://docs.aws.amazon.com/AmazonS3/latest/userguide/glacier-storage-classes.html) is another cold storage option designed for more aggressive cost savings. It offers very low storage costs but requires a restore request before the file can be downloaded. Depending on the Glacier tier, this restore process can take from several minutes to several hours.

These transitions don’t need to be managed manually by the backend. Instead, they can be automated using AWS S3’s lifecycle policies. For example, a rule like “move to Glacier 30 days after upload” can be applied at the object level, helping to reduce storage costs without increasing operational overhead.

That said, cold storage does introduce latency when restoring files for download. Options like `S3 Glacier Deep Archiv`e offer the lowest cost, but the long restore time can significantly affect the user experience. On the other hand, `S3 Glacier Instant` is the most expensive within the Glacier family but provides restore times in the milliseconds to seconds range. For a service like MY BOX, where availability matters, Glacier Instant may be the most reasonable option.

#### Summary

Building a system that can reliably handle large file uploads and downloads—while also considering long-term storage costs—involves more complexity than it may seem. In this post, we walked through a system design for a hypothetical file storage service called MY BOX, covering everything from high-level flow to low-level implementation, with support for both small files and large files up to several dozen gigabytes.

Of course, this is not the end. Features like file sharing, version control for real-time collaboration, and others we left out of scope here could be explored in future extensions. Topics like security, data deletion policies, and storage redundancy also offer plenty of depth for further design work.

This was a system design exercise built around MY BOX, and hopefully, I’ll return with another topic in the future.
