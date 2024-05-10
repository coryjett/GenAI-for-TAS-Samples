# UAA performance
This topic provides an overview of the configuration and results of performance tests conducted on the UAA API. It does not discuss log outputs or metrics.
For information about monitoring and diagnosing UAA performance over time, see [UAA performance metrics](https://docs.cloudfoundry.org/uaa/uaa-metrics.html).
The performance test results illustrate the baseline performance of a single instance and expected performance when horizontally scaling the number of UAA virtual machine (VM) instances.
The graphs in this topic are presented as time series and describe the effect of increasing concurrency on throughput and latency. Operators can use this information to determine the number of instances required to achieve a target throughput under reasonable latency.

## Benchmarking
UAA testing for this topic was performed in the context of a Cloud Foundry environment instead of in full isolation tests.
The benchmarking test setup places the database, client, router, and UAA servers within the same network to minimize external network factors on performance. The test setup relies on Gorouter to load balance requests across multiple UAA instances.
The [Apache JMeter](http://jmeter.apache.org/) client increases step concurrency to reach the target currency for a given test run. The testing tools are packaged as a BOSH release and deployed to a dedicated VM. The performance tests exercise the following flows:

* **Authorization code** /oauth/authorize and /oauth/login flow

* **Client credentials** /oauth/token flow

* **Implicit** /oauth/authorize flow

* **Password** /oauth/token flow
Each request generated by JMeter is:

* Sent to the router HTTPS protocol

* Includes UAA URL as Host Header (used by Gorouter to decide the host)

* Includes client credentials as a URL parameter

* For authorization code and implicit grant types, an initial login is performed and JSESSIONID is passed for subsequent authorize and token requests

* For password grant type, user credentials are included as URL parameters
When JMeter receives a response from UAA, it:

* Validates response code to ensure request is successful.
JMeter stores each request entry to in an intermediate JTL(CSV) format file and this is used as an input to generate graphs.
We simulate increasing load by ramping up the number of concurrent threads in increments of 20 threads until we reach the equivalent of 200 threads per VM. This allows us to observe the behavior of UAA as the load increases to give us a better understanding of how UAA behavior changes as load increases.

## Tools
[JMeter test plan](https://github.com/cf-identity/jmeter): A collection of JMeter scripts. UAA testing for this topic uses these scripts with JMeter to load test, measure, and analyze performance. JMeter supports variable parameterization, response assertions and validation, and graph generation.

### Benchmarking setup

* VM Instance Properties

+ GCP Cloud Config Name: n1-standard-2

+ GCP Machine Type: n1-standard-2

+ CPU Information: 2 x Intel® Xeon® CPU @ 2.60GHz

+ RAM Information: 7479 MB

+ UAA Max Heap Size: 768 MB

+ UAA Version: `v53.1` branch of UAA

* Database Instance Properties

+ GCP Cloud Config Name: n1-standard-2

+ CPU: 2 x Intel® Xeon® CPU @ 2.60GHz

+ RAM Information: 7479 MB

+ Allocated Storage: 10Gb of Persistent Disk

+ Engine: MySQL

* Network Setup:

+ MySQL deployed with CF deployment so the connection from UAA to database within the same network

+ GCP does not document any network bandwidth on any vm types

+ Gorouter is scaled to 8 GB memory to prevent router bottlenecks

* Data Configuration: Test dataset includes the following:

+ 1 client

+ 1 User

+ 41 Groups

## Performance results

### Client credentials grant type

**Endpoint**: /oauth/token?grant\_type=client\_credentials
| **Instances** | **Threads** | **Throughput** |
| --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/client-creds-threads-1.png) | [Throughput Level 1](https://docs.cloudfoundry.org/running/images/client-creds-throughput-1.png) |
| 2 | [Threads Level 2](https://docs.cloudfoundry.org/running/images/client-creds-threads-2.png) | [Throughput Level 2](https://docs.cloudfoundry.org/running/images/client-creds-throughput-2.png) |
| 4 | [Threads Level 4](https://docs.cloudfoundry.org/running/images/client-creds-threads-4.png) | [Throughput Level 4](https://docs.cloudfoundry.org/running/images/client-creds-throughput-4.png) |
| **Instances** | **Threads** | **Latency** |
| --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/client-creds-threads-1.png) | [Latency Level 1](https://docs.cloudfoundry.org/running/images/client-creds-latency-1.png) |
| 2 | [Threads Level 2](https://docs.cloudfoundry.org/running/images/client-creds-threads-2.png) | [Latency Level 2](https://docs.cloudfoundry.org/running/images/client-creds-latency-2.png) |
| 4 | [Threads Level 4](https://docs.cloudfoundry.org/running/images/client-creds-threads-4.png) | [Latency Level 4](https://docs.cloudfoundry.org/running/images/client-creds-latency-4.png) |
Server Performance metric like CPU utilization, heap memory usage, and time
spent in garbage collection. These metrics are collected by an agent colocated
with the UAA VM. Graphs are plotted as time series.
| **Instances** | **Threads** | **CPU** | **Heap Memory** | **Garbage Collection** |
| --- | --- | --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/client-creds-threads-1.png) | [CPU Level 1](https://docs.cloudfoundry.org/running/images/client-creds-cpu-1.png) | [Heap Memory Level 1](https://docs.cloudfoundry.org/running/images/client-creds-heap-1.png) | [Garbage Collection Level 1](https://docs.cloudfoundry.org/running/images/client-creds-garbage-1.png) |
| 2 | [Threads Level 2](https://docs.cloudfoundry.org/running/images/client-creds-threads-2.png) | [CPU Level 2](https://docs.cloudfoundry.org/running/images/client-creds-cpu-2.png) | [Heap Memory Level 2](https://docs.cloudfoundry.org/running/images/client-creds-heap-2.png) | [Garbage Collection Level 2](https://docs.cloudfoundry.org/running/images/client-creds-garbage-2.png) |
| 4 | [Threads Level 4](https://docs.cloudfoundry.org/running/images/client-creds-threads-4.png) | [CPU Level 4](https://docs.cloudfoundry.org/running/images/client-creds-cpu-4.png) | [Heap Memory Level 4](https://docs.cloudfoundry.org/running/images/client-creds-heap-4.png) | [Garbage Collection Level 4](https://docs.cloudfoundry.org/running/images/client-creds-garbage-4.png) |

### Password grant type

**Endpoint**: /oauth/token?grant\_type=client\_credentials
| **Instances** | **Threads** | **Throughput** |
| --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/password-grant-threads-1.png) | [Throughput Level 1](https://docs.cloudfoundry.org/running/images/password-grant-throughput-1.png) |
| 2 | [Threads Level 2](https://docs.cloudfoundry.org/running/images/password-grant-threads-2.png) | [Throughput Level 2](https://docs.cloudfoundry.org/running/images/password-grant-throughput-2.png) |
| 4 | [Threads Level 4](https://docs.cloudfoundry.org/running/images/password-grant-threads-4.png) | [Throughput Level 4](https://docs.cloudfoundry.org/running/images/password-grant-throughput-4.png) |
| **Instances** | **Threads** | **Latency** |
| --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/password-grant-threads-1.png) | [Latency Level 1](https://docs.cloudfoundry.org/running/images/password-grant-latency-1.png) |
| 2 | [Threads Level 2](https://docs.cloudfoundry.org/running/images/password-grant-threads-2.png) | [Latency Level 2](https://docs.cloudfoundry.org/running/images/password-grant-latency-2.png) |
| 4 | [Threads Level 4](https://docs.cloudfoundry.org/running/images/password-grant-threads-4.png) | [Latency Level 4](https://docs.cloudfoundry.org/running/images/password-grant-latency-4.png) |
Server Performance metric like CPU utilization, heap memory usage, and time
spent in garbage collection. These metrics are collected by an agent co-located
with the UAA VM. Graphs are plotted as time series.
| **Instances** | **Threads** | **CPU** | **Heap Memory** | **Garbage Collection** |
| --- | --- | --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/password-grant-threads-1.png) | [CPU Level 1](https://docs.cloudfoundry.org/running/images/password-grant-cpu-1.png) | [Heap Memory Level 1](https://docs.cloudfoundry.org/running/images/password-grant-heap-1.png) | [Garbage Collection Level 1](https://docs.cloudfoundry.org/running/images/password-grant-garbage-1.png) |
| 2 | [Threads Level 2](https://docs.cloudfoundry.org/running/images/password-grant-threads-2.png) | [CPU Level 2](https://docs.cloudfoundry.org/running/images/password-grant-cpu-2.png) | [Heap Memory Level 2](https://docs.cloudfoundry.org/running/images/password-grant-heap-2.png) | [Garbage Collection Level 2](https://docs.cloudfoundry.org/running/images/password-grant-garbage-2.png) |
| 4 | [Threads Level 4](https://docs.cloudfoundry.org/running/images/password-grant-threads-4.png) | [CPU Level 4](https://docs.cloudfoundry.org/running/images/password-grant-cpu-4.png) | [Heap Memory Level 4](https://docs.cloudfoundry.org/running/images/password-grant-heap-4.png) | [Garbage Collection Level 4](https://docs.cloudfoundry.org/running/images/password-grant-garbage-4.png) |

### Authorization code grant type

**Endpoint**: /oauth/authorize?grant\_type=authorization\_code and /oauth/token?grant\_type=authorization\_code
| **Instances** | **Threads** | **Throughput** |
| --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/auth-code-grant-threads-1.png) | [Throughput Level 1](https://docs.cloudfoundry.org/running/images/auth-code-grant-throughput-1.png) |
| **Instances** | **Threads** | **Latency** |
| --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/auth-code-grant-threads-1.png) | [Latency Level 1](https://docs.cloudfoundry.org/running/images/auth-code-grant-latency-1.png) |
Server Performance metric like CPU utilization, heap memory usage, and time
spent in garbage collection. These metrics are collected by an agent colocated
with the UAA VM. Graphs are plotted as time series.
| **Instances** | **Threads** | **CPU** | **Heap Memory** | **Garbage Collection** |
| --- | --- | --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/auth-code-grant-threads-1.png) | [CPU Level 1](https://docs.cloudfoundry.org/running/images/auth-code-grant-cpu-1.png) | [Heap Memory Level 1](https://docs.cloudfoundry.org/running/images/auth-code-grant-heap-1.png) | [Garbage Collection Level 1](https://docs.cloudfoundry.org/running/images/auth-code-grant-garbage-1.png) |

### Implicit grant type

**Endpoint**: /oauth/token?grant\_type=client\_credentials
| **Instances** | **Threads** | **Throughput** |
| --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/implicit-grant-threads-1.png) | [Throughput Level 1](https://docs.cloudfoundry.org/running/images/implicit-grant-throughput-1.png) |
| **Instances** | **Threads** | **Latency** |
| --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/implicit-grant-threads-1.png) | [Latency Level 1](https://docs.cloudfoundry.org/running/images/implicit-grant-latency-1.png) |
Server Performance metric like CPU utilization, heap memory usage, and time
spent in garbage collection. These metrics are collected by an agent colocated
with the UAA VM. Graphs are plotted as time series.
| **Instances** | **Threads** | **CPU** | **Heap Memory** | **Garbage Collection** |
| --- | --- | --- | --- | --- |
| 1 | [Threads Level 1](https://docs.cloudfoundry.org/running/images/implicit-grant-threads-1.png) | [CPU Level 1](https://docs.cloudfoundry.org/running/images/implicit-grant-cpu-1.png) | [Heap Memory Level 1](https://docs.cloudfoundry.org/running/images/implicit-grant-heap-1.png) | [Garbage Collection Level 1](https://docs.cloudfoundry.org/running/images/implicit-grant-garbage-1.png) |