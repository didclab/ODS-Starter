# ODS-Starter

A repository detailing the documentation for the OneDataShare System

This document will discuss the various deployment patters that users can take to moving their data. It will also cover
the protocols we support as well as the limitations & enhancements we apply per protocols.

## Disclosure

If you have any feature requests, questions, or concerns please feel free to open an issue in this repository, and we
will answer promptly. Cheers

## The OneDataShare Repositories

1. [ODS CLI](https://github.com/didclab/ods-cli) <sub>(Release)</sub> a cli that interacts with the ODS core to provide a replacement to the UI
2. [Pmeter](https://github.com/didclab/pmeter) <sub>(Release)</sub> an open source CLI tool developed to monitor the network, and kernel parameters of the host running the Transfer-Service
3. [ODS Transfer Service](https://github.com/didclab/Transfer-Service) <sub>(Release)</sub> the actual data transfer service that utilizes threading to push your data.
4. [ODS Transfer Scheduler](https://github.com/didclab/transfer-scheduler) <sub>(Release)</sub> a publisher to RabbitMQ that allows your Transfer-Job Requests to be queued.
5. [ODS MetaData](https://github.com/didclab/ods-metadata) <sub>(In Development)</sub> a service that enables the querying of all the data transfers you have submitted as a user
6. [Endpoint Credential](https://github.com/didclab/endpoint-cred-service) <sub>(Release and Private)</sub> the service that stores and manages endpoint credentials
7. [C++ SDK](https://github.com/didclab/CClient) <sub>(Beta)</sub>
8. [OneDataShare Core](https://github.com/didclab/onedatashare) <sub>(Release)</sub> the core service running onedatashare.org

## Supported Protocols

- [S3](https://www.researchgate.net/publication/335608365_Amazon_S3) <sub>(Release)</sub>
- [FTP](https://www.rfc-editor.org/info/rfc959) <sub>(Release)</sub>
- [SFTP](https://datatracker.ietf.org/doc/html/rfc913) <sub>(Release)</sub>
- [HTTP](https://datatracker.ietf.org/doc/html/rfc2616) <sub>(Release)</sub>
- VFS stands for virtual file system and is only available to users who use the Hybrid or On-Premise deployment
  model <sub>(Release)</sub>
- [Box](https://www.box.com/) <sub>(Release)</sub>
- [Dropbox](https://www.dropbox.com/) <sub>(Release)</sub>
- [SCP](https://en.wikipedia.org/wiki/Secure_copy_protocol) <sub>(Release, SCP is deprecated as of April 2019 and uses SFTP by default)</sub>
- [Google Drive](https://www.google.com/drive/) <sub>(Beta)</sub>


## Use Case

Let's discuss the use case of OneDataShare and why users should activley consider ODS compared to projects such as: 
[RClone](https://github.com/rclone/rclone), [Globus](https://www.globus.org/), [Airavata](https://arxiv.org/pdf/2107.03882.pdf), [Go Anywhere](https://www.goanywhere.com/solutions/cloud-file-transfer) tool. 
Consider the following image ![Tweet](images/CUBoulderTweet.png)
Let's consider a few cases as to how one might move 2TB over different networks using a WAN or more generally some kind of long fat network.

1. My personal ISP is Spectrum and currently I am purchasing a 400Mbps network, and I have an RTT of ~60ms with an MTU of 1500.

In this case the minimum transfer time I can hope to accomplish is 11 hours and 42 seconds. Which is reasonable?
   
2. Now lets consider what an HPC can bring to end users. Every node in Chameleon Cloud has a 10Gbps defined through SDN, which then connects to a switch that has a 40Gbps link. With an RTT of ~50ms and a MTU of 1500.
If we were to begin the transfer lets assume it is not possible to go past 10Gbps due to the SDN(in reality this is not true), this means we can hope to accomplish a minimum transfer time of 1 hour and 54 seconds which is a massive increase in throughput but again it is still an unreasonable amount of time.
   

If ODS did not exist I would most likely use RClone to transfer my data from Google Drive to OneDrive as my goal would be to get my data across as quickly as possible.
The big issue is that RClone is static when it comes to observing your network it does an initial check and then uses those parameters for the rest of the transfer offering no dynamic tuning depending on the conditions of your network.
Networks are highly volatile, and they are most importantly fair. This means that there is a local maximum throughput changing constantly, and there is a long list of factors affecting your experience on networks like WAN's LFN's.
OneDataShare has the sole goal of moving your data as fast as possible, while maintaining security standards and not storing any of your data in our system.
If you would like to read more about the research behind ODS I have provided links below to relevant research papers.

## SaaS Users

[OneDataShare](https://onedatashare.org) for SaaS users is very simple. A user can create a free account and add any of
the supported protocols using the frontend. This will allow users to navigate and manage their filesystems through the
OneDataShare UI. This approach is very supportive of users who do not wish to install anything on their system or are
simply unable too. This is the simplest approach to using OneDataShare as there is nothing, but a UI. When the user has added at
least 2 endpoints then the user may begin sending file transfers between the two endpoints. We currently do not support
the scattering and gathering of files. If this is a feature you require please submit an issue with a clear description
of how and why you would like this operation. In the bottom of the Transfer-Page the user is able to select some
initial parameters to begin the transfer with. Now each protocol has limitation which will be covered below as these are
hard restrictions.

## Transfer Options

This section will explain the verbiage of the transfer options, and some recommended default values.

- Parallelism: the number of parallel threads to transfer a single file. The range I would advise if supported is 1-64.
  I do not believe that anything above 64 would do anything but add a massive amount of context switching.
- Concurrency: the number of files to transfer simultaneously and in parallel. This is limited to the number of
  connections the receiving server is able to open for a single user.
- Chunk Size: The buffer size that each thread should read and then write for every file in the transfer. This value is
  fixed throughout the entire transfer, and I would advise to be aggressive with this value. The higher the value the
  more abusive this will be on RAM as such decreasing the number of IO operations performed.
- Pipe Size: This represents pipelining, and it is the number of successive read() operations to a single write()
  operation. If you come from a batch systems world then this would be called commit interval as well. The longer the
  distance between the source and destination the higher this value should be.
- Retry: A fixed number of times to retry a chunk per file. So if this is set to 5, and a file has had 5 failures in
  sending any chunk of the file. Then after 5 retries we will fail that fail and continue with the transfer.
- Overwrite: If the destination has the file with the same name/id in the same path then we will over write that file.
- Encrypt: This value is currently meaning less, but if set to true we will then use SSL -> FTPS, SFTP, HTTPS.
- Compress: Currently this only supported for SFTP as this is native to the protocol. ODS decides the compression level.
- Optimization: We are currently developing the Optimization side of the project, right now we plan to add two kinds:
  Reinforcement Learning, and Bayesian Optimization to tune the transfers in real time.

Currently, ODS uses [t2.medium](https://aws.amazon.com/ec2/instance-types/)
and [C4](https://aws.amazon.com/ec2/instance-types/) instances to do your data transfer. As we do not have any payment
system, if you require larger instances please submit and instance, and we will work with you for the desired result.

## Protocol Limitations Table

This table represents what ODS currently supports, we are working on supporting everything below. 

| Protocol | Parallelism | Concurrency | Pipelining | Chunk Size | Optimizer | Retry | Compress | Integrity | Overwrite | Read | Write |
| -------- | ----------- | ----------- | ---------- | ---------- | --------- | ----- | -------- | --------- | --------- | ---- | ----- |
| S3 | &check; | &check; | &check; | > 5MB | &check; |&check; |&check; |&cross; | &check; |&check; | &check; |
| FTP | &cross; | check max supported connections per user | &check; | &check; | &check; |&check; |&cross; |&cross; | &check; | &check; | &check; |
| SFTP | &cross; | check max supported connections per user | &check; | > 5MB | &check; |&check; |&check; |&cross; | &check; | &check; | &check; |
| HTTP | &check; | &check; | &check; | &check; | &check; |&check; |&cross; |&cross; | &cross; | &check; | &cross; |
| Box | &check; | &check; | &check; | 20MB | &check; |&check; |&check; |&check; | &check; | &check; | &check; |
| Dropbox | &check; | &check; | &check; | &check; | &check; |&check; |&cross; |&check; | &check; | &check; | &check; |
| Google Drive | &cross; | &check; | &check; | &check; | &check; |&check; |&cross; |&cross; | &check; | &check; | &cross; |

## Hybrid Users

I would like to state if you are a "power user" or someone in the HPC world that is attempting to move massive data over a WAN or really any kind of network
then I would advise you use this deployment if your HPC site gives you access to install and deploy your own
software. Many HPC sites such as [CCR](https://ubccr.freshdesk.com/support/solutions/articles/13000014882-data-transfer-options) offer a DTN(Data Transfer Node) that has a very high capacity link to the public internet. 
The DTN is where I would recommend deploying the [Transfer-Service](https://github.com/didclab/Transfer-Service) and I would
recommend you deploy this on your site using Singularity because Docker has a software defined networking which would provide
an overhead to the network I/O. We expose a Dockerfile that you can convert into a Singularity build. I am currently
writing a singularity configuration file, so we will soon offer a direct singularity build for all customers to use. 
If your site does not offer any container runtime then you will need to manually install, to do this please reference the [README.txt](https://github.com/didclab/Transfer-Service).

## On-Premise

Deploying our entire ODS platform onto your site is not something I would advise but if you have a valid use case please
open a ticket, and we will be in contact promptly.

## ODS Stack
- [Java 11](https://openjdk.java.net/projects/jdk/11/)
- [Python 3.0+](https://www.python.org/download/releases/3.0/)
- [Spring](https://spring.io/)

### Spring
- [Spring Batch](https://spring.io/projects/spring-batch)
- [Spring Cloud](https://spring.io/projects/spring-cloud)
- [Spring Data](https://spring.io/projects/spring-data)
- [Spring Security](https://spring.io/projects/spring-security)
- [Spring AMPQ](https://spring.io/projects/spring-amqp)
- [Spring Boot](https://spring.io/projects/spring-boot)
- [Spring Reactive](https://spring.io/reactive)

### Databases
- [DocumentDB](https://aws.amazon.com/documentdb/) stores your OneDataShare login credentials which provides default up to date security standards from Amazon.
- [CockroachDB](https://www.cockroachlabs.com/) provides the ability to store metadata regarding your transfer, so we can restart any file that fails as well as optimize future transfers from your past transfer.
- [Vault](https://www.hashicorp.com/products/vault) stores the endpoint credentials you add to OneDataShare.
- [InfluxDB](https://www.influxdata.com/) an open source time series database that allows us to monitor active transfer-services

## ODS Research Links
- [Application-Level Optimization of Big Data Transfers through Pipelining, Parallelism and Concurrency](https://ieeexplore.ieee.org/abstract/document/7065237)
- [High-Speed Transfer Optimization Based on Historical Analysis and Real-Time Tuning](https://ieeexplore.ieee.org/document/8249824)
- [Big Data Transfer Optimization Through Adaptive Parameter Tuning](https://par.nsf.gov/servlets/purl/10074013)
- [Prediction of Optimal Parallelism Level in Wide Area Data Transfers](https://ieeexplore.ieee.org/abstract/document/6018962)
- [End-to-End Data-Flow Parallelism for Throughput Optimization in High-Speed Networks](https://link.springer.com/article/10.1007/s10723-012-9220-9)
- [Dynamically tuning level of parallelism in wide area data transfers](https://dl.acm.org/doi/abs/10.1145/1383519.1383524)
- [Energy-Efficient Data Transfer Optimization via Decision-Tree Based Uncertainty Reduction](https://arxiv.org/abs/2204.07601)

# Per Service Documentation
In this section of the document I would like to discuss how each service works, and the goals/responsibilities that it fulfills. 
If you wish to look at more specific things than I would advise looking at each repository independently and learning how ODS implemented it.
In the case the reader wishes to understand how to deploy or interact with any specific service this information can and will be found under that repositories README.md.

## ODS Core
Back in 2019 when I first came to ODS the project was fully covered using this repository. It handled user management, credential management, data transfer specifics, and overall was the true definition of a core. 
The project decided to migrate towards a micro-services architecture which meant that this service, was designated the task of handling all user related requests. 
I find this to not be very interesting, nonetheless it is important to understand that this service is tasked with being the service that faces users, and handles: hosting frontend, providing access to all user related data. In the essence it is the service that all users interact with.

## Endpoint Credential Service
This service is a private repo, so you as a non-developer are not allowed at all to access the source code or understand the API of this service. The reason we did this is b/c we want to prevent any information from being exposed as to how we encrypt and store your information at rest.
What I can tell you is that we use Vault, and DocumentDB to store all endpoint-credentials. Examples of such things are: pem files, basic auth, oauth tokens,,,, etc. The access to this service is strictly inter-service and the only time I can image a user wanting to access this service is when they are following the on-premise deployment which we still have not currently fleshed out entirely.
Fundamentally this service is the only layer of access to your credentials, meaning no other service can access where your credentials are resting. A further layer of security is that only IP addresses from inside our VPC and not even all of those are allowed to access your credentials, we take the security of your credentials very seriously and if you have any recommendation on how we can improve or general questions please feel free to ask or comment, and we will answer promptly and transparently.

## Transfer Scheduler
The Transfer-Scheduler is only used when you are submitting transfer jobs. It is a publisher to RabbitMQ, and we do validation at this layer. We collect some metadata regarding your transfer request: find the file sizes, ensure those files exist, and expand the file paths, while adding a priority to your request. Currently, every user has the same priority as we have no differentiation between users.
The idea of this service is to do validation checks on your request to make sure that it is possible before we publish the request.

## Transfer Service
The Transfer Service is a consumer to RabbitMQ and does the data transfer.
During the data transfer we send the statistics of the transfer service such as: memory usage, cpu usage/frequency, throughput,,, etc to InfluxDB for monitoring. In this way we are able to monitor transfers in real time.
So how does Real time Transfer Optimization come into play? During the Transfer per interval we reach out to an optimizer submitting the throughput and corresponding transfer options, and we then receive the values to apply all in a non-blocking fashion. 
Thus, your transfer will never be delayed or blocked, and will dynamically change as it progresses.
This service acts as the client between both the source and destination and nothing ever gets written to disk, everything is handled as byte arrays and that is intentional. We do not add any interpretation to your data nor do we want to as that would decrease performance of the transfer.
So a question that naturally occurs is, if we do not apply any transformations to your data how do things like encryption, compression, and check summing work in our system?
To put it simply we do not apply this ourselves, we rely on the protocol. So S3 for example supplies a checksum to a file, but it does not compress your data natively. If you the user submitted a request that had compression flagged for S3 then we would add a header "gzip" with the compression level to all the requests for that file. Then S3 would service the file compressed to that level and thus we have compression.
Checksum is the only option we compute ourselves and this is done using the MessageDigest class exposed in Java. So every chunk of data that we download or upload we add to the digest and when we have finished operating on that file we digest the data in the MessageDigest class and then finally compare the checksum the Transfer Service calculated to the checksum exposed by the source or destination. In the case the digest is wrong we re-do the entire file transfer as it is impossible to tell which chunk was not sent correctly.


To the final most important part of this service, Optimization.
I would like to make it clear that optimization is only useful when you have a large transfer request and if you are sending anything ~<1GB it is almost entirely useless as your transfer will complete very quickly most likely faster than the interval we used to communicate with the optimizer service.
To re-state we have or are developing/testing two different kinds of optimization services. 


1. Bayesian Optimization: This is implemented and tested but not yet deployed as I have not had the time too as of late. In the essence in an interval we reach out to the optimizer with the following input parameters: concurrency, pipelining, parallelism, throughput values. 
   Following the optimizer responds with the next parameters to use, and we apply those to the thread pools and clients which do not stop the transfer. Just an FYI but in Java the thread pools we use is a class called [ThreadPoolExecutor](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ThreadPoolExecutor.html), what I have observed is that whenever we change the size of the pools it does not happen instantly. Only when demand of work increases on the pool will the pool increase its thread count to the new size set.
   I have not found this explicit in the documentation that I have linked, but it is merely an observation. Currently, we query the BO optimizer every 2 min which I have seen to be enough time for the thread values to change and for us to see a rise or fall in throughput. Remember BO is not some magic recipe to faster throughput but in long transfers it will converge and keep adapting the network condition as its goal is to maximize throughput of the data transfer.  
 
2. Reinforcement Learning: I will leave this blank as this is something DIDCLAB is actively researching.

## Metadata Service
This service is responsible for querying CockroachDB and exposes the meta data the ODS system has generated for current and past transfer jobs per user. 
The metrics we store follow along these lines: start/end time per job, per file, read/write counts, skipped files, number of errors, the parameters of job, the step parameters,, etc. This meta data enables the system to retry automatically, and is largely provided by the Spring Batch Framework.
This is not yet a finalized service, so I will end it here.

## Pmeter

Pmeter is an open source tool that was developed in CSE 603 under Dr. Jaroslaw Zola any relevant documentation will be found in that repository as it is not strictly an ODS tool.

## ODS CLI

The ODS CLI is a replacement identical in functionality to the UI except it does not provide the ability to submit tickets or change passwords. This developed to serve the purpose of allowing "power users" to not have dealt with UI and simply work out of terminal.
As the author this is my preferred way of interacting with the system, the reason being is it feels much more fluid. Working with the UI tends to be buggy due to the nature of developing user-friendly UI's is an entirely separate challenge. Hence the CLI serves the same fundamental service.
The interactions one can find in the UI are supported by the CLI. This means once a user has created an account and added their JWT token to the cli the user is able to use the cli as they would the UI.
