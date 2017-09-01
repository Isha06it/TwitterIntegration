# TwitterIntegration
Introduction:

This project is used to do the integration between Twitter REST API and Database using TIBCO BW. Need to fetch data using Twitter Search API and storing specific data to database (as per requirement).

Requirements:

Solution should query Twitter REST API, enriches the data with a "tag" field set with a value of "IoT" and publishes the data to consumers. 
There is currently only one consumer ("acme") of this service that requires the new releases to be sync'd into a database with fields "id","text","tag". acme is expected to be an old system and is not able to process records at the rate that they are published. Please limit the number of payload to 10 items per request. You are required to build and simulate the acme  system. The integration must support the requirement for acme to be offline if required.

Technology & Tools:

TIBCO BW 5.11
TIBCO EMS
TIBCO REST JSON Plugin
Oracle 11g Database

Description of Project:


Following Process Definitions are created:

1. Invoke Twitter : To extract the data from Twitter REST API.
2. Database : To insert data into database.
3. PUB Framework: To publish data on Topic.
4. Error Handler: TO record the error.

Invoke Twitter: 

This process definition's purpose is to extract data with tag "Iot" and send it to PUB Framework. A timer is set for every 1 minute to run this process and Invoke Rest API. REST API returns JSON data for the tag "Iot". There is a condition to restrict the search to return only 10 records at a time . Extract the required fields "id", "text" and "tag" and send it to PUB Framework to publish it to Acme database.
A catch activity is also there to catch any kind of exception and send it to Error Handler with message and message code.

PUB Framework:

This is a generic framework to publish data on Topic. This process requires the topic name and data as input. It then sends the data using JMS Topic Publisher activity to database process where the Subscriber activity is listening for messages.
Using this frameworkwe can add "n" number of subscribers listening to the topic for data.
A catch activity is also there to catch any kind of exception and send it to Error Handler with message and message code.

Database:

This process listens to the topic using JMS Topic Subscriber Activity. Once the message is received JDBC Update Activity sends the data to Acme database. At a time only 10 records are inserted. 
If the Acme database is Offline this process keeps on retrying to send the data . Retry happens in every 30 sec. Once the database comes online all the messages pending till now will be inserted in database.
A catch activity is also there to catch any kind of exception and send it to Error Handler with message and message code.

Error Handler:

This process is to log the error message and code. Currently this procees doesn't do much, but considering the future enhancements separate process helps to manage the error handling in a better way.

--> All the variables are fetched from GLobal variable list, hard coding in the development was avoided. Making the code configurable helps to reduce the deployment cycles.

--> Single Schema was used throughout project for fields "id" , "text" and "tag".

Unit testing:


Prerequisite for running the code:

1. EMS server should be up and running.
2. Create the Topic: "twitter" using the command "create topic twitter" at EMS console.
3. Create Topic and Queue connection Factory using following commands:

create factory QueueConnectionFactory queue url=tcp://localhost:7222

create factory TopicConnectionFactory1 topic url=tcp://localhost:7222

4. Test the JMS connection.
5. Table with name "Twitter" should be connected in Oracle database.
6. Columns of table: ID (Varchar 50), TEXT (VARCHAR 1000), TAG (VARCHAR 200)
7. The Oracle database should be up and running.
8. Test the JDBC connection.

Run the code:

1. In TIBCO Designer, open the project twitter.
2. Click the Tester tab and then all the processes from Business Process folder.
3. Right click Invoke Twitter process definition in tester tab. Select Create job.
4.The activities and transition lines are highlighted in green as incoming information is processed.

Process Flow: Success Scenario

After Starting "Invoke Twitter" process definition, the Timer will start every minute to invoke rest Api. The Api returns the data in JSON format. After formatting the JSON data it will be send to "PUB Framework" process, which publishes the data on Topic "twitter. "Database" process subscribes the data and sends it to database.

Use Cases:

1. Success Scenario: After the proceess finishes in TIBCO designer, check the database new rows are added with the data.
2. EMS Server Down: In this scenario in PUB Framework process the JMS activity will not be able to send the data and fails. It gives exception which then cathed by Error Handler.
3. Database is down: In this scenario when request reaches "database" procees, the JDBC activity fails. But it will retry till the data is inserted properly. If more than 1 job is created till the database comes online, all the data from all jobs will be inserted. Retry occurs in every 30 secs.
4. Rest API fails: In this scenario the first process "Invoke twitter" fails and flow goes to Error Handler.

Future enhancements:

New Subscribers can be eaily added without changing the Invoke Twitter, PUB Framework and Error Handler. This solution is based on service design principles. All the process definations are independent to each other. New API can be added easily without changing other code.

Alternative options:

Tried using Mulesoft to implemet the same. I was able to get the JSON data and save it file after formatting.
