# URL Shortener

![bitly image](https://cdn2.hubspot.net/hub/431273/file-3019062849-jpg/blog-files/bitly-logo.jpg)

## System Design Challenge

Design a url shortening service like [bitly.com](http://www.bitly.com). 

If you're new to system design problems or are not sure where to start, check out the Resources section below.

This example is taken from [Hired In Tech](https://www.hiredintech.com/classrooms/system-design/lesson/55).

## Clarifying Questions

If you're working through this problem by yourself or doing a mock interview with a friend, these are the answers that the interviewer would give to the following questions:

*Q: Do I need to design for any use cases besides url shortening and redirection?*  
*A: No, the only functionality that you need to worry about is shortening urls and redirecting the user to the proper url when they input a shortened url.*

*Q: How many requests per month does the system need to be able to handle?*  
*A: Assume that it's not going to be in the top 3 url shortening services, but it will be in the top 10.*  
OR   
*A: The system should be able handle 1 billion requests per month.  100M requests per month are for shortening new urls and the remaining 900M requests per month are for redirection.*

*Q: Do we know the read/write ratio of our monthly traffic?*  
*A: You can assume that 10% of requests are writes and 90% are reads.*

## The Solution
  
### Step 1: Constraints and Use Cases

#### Use Cases ####

The first thing that you want to do is ask the interviewer questions in order to clarify the use cases for your system.  For our Bitly clone we're going to have two main use cases.

1. URL Shortening - recieve a url from the user and return a short url
2. Redirection - recieve a request to a short url and redirect to the original url

We don't need to dive into the specifics of how these will work yet, but you should have a general idea of how we'll handle each use case.  For the url shortening, we'll have a hashing function that will take a url and return back a hash value. We'll store this hash value and url in a data store as a key-value pair and use the hash value as a short url. When we recieve a request to a short url, we'll query the data store for the long url and return a redirect. 

Other potential use cases that you could ask about include:
  * Analytics
  * Automatic link expiration
  * Manual link removal
  * UI vs. API

#### Constraints ####

The next thing that you want to do is clarify the constraints.  These are the factors that you will need to take into consideration as you design your system.  The constraints that we'll start with are load and data.

##### Load #####
You need to figure out how much traffic your system will be required to handle.  For this example, a good breakdown of monthly traffic would be:
  * Total number of requests per month
  * Number of requests that are for url shortening (writes)
  * Number of requests that are for redirection (reads)


At this point you can ask your interviewer directly for this information.  They may tell it to you or they may expect you to figure out a ballpark number.

*Q: How many requests per month does the system need to be able to handle?*  
*A: Assume that it's not going to be in the top 3 url shortening services, but it will be in the top 10.*

Since Twitter is one of the main drivers of short url usage, a good way to come up with an estimate for your monthly traffic is to base your numbers off of Twitter's monthly traffic. 

We know that Twitter users generate 500 million tweets/day => 15B tweets/month  
Let's assume that 5-10% of tweets use a shortened url  
That would mean the total number of new urls being shortened each month is roughly 1.5B  
Let's assume that 80% of requests go to the top 3 link shortener sites => 1.2B  
The remaining 20% of requests go to all of the other link shortern sites => 300M   
If we're one of the top 10 link shortener sites, but not one of the top 3, a conservative estimate for the number of links that we shorten a month would be 100M

Now that we know how many writes per month (url shortening) we'll be handling, we need to figure out how many reads per month (redirect requests) we need to design for.  Again, you can ask your interviewer for this information directly and they may tell you or they may expect you to come up with a reasonable answer yourself.

*Q: Do we know the read/write ratio of our monthly traffic?*  
*A: You can assume that 10% of requests are writes and 90% are reads.*

Total: 1B req/month => 400 req/sec  
Writes: 100M req/month => 40 req/sec   
Reads: 900M req/month => 360 req/sec  

##### Data #####
Now that we know how much traffic our system is going to need to be able to handle, we need to get an idea of what the data for our system is going to look like.  There are two questions that we'll be trying to answer that will help us understand our data contraints:

  * How much data will we need to store?
    * Size of the urls
    * Size of the hash values
  * How much data will be passing through our system at any given time?
    * Reads
    * Writes

Let's assume that we'll be desiging the system to be able to store 5 years worth of urls  
Total urls: 5 years * 12 months/year * 100M urls/month = 6B urls  
let's assume that the average length of a url is 500 characters  
1 char = 1 byte  
500 bytes per URL => 3TB for all urls  

Since we have 6B urls, we will need 6B unique hash values   
We will use Base62 for our hash values (numbers, uppercase letters, and lowercase letters)  
5 character long hash values => 62^5 = ~1B unique combinations  
6 character long hash values => 62^6 = ~57B unique combinations   
Since we need 6B unique combinations, each hash value will need to be 6 characters long   
1 char = 1 byte  
6 bytes per hash value => 36GB for all hash values  

Now we can figure out how much data our system will be moving around at any given time  
Data written per second: 40 writes * (500 + 6) bytes = ~20kb/sec  
Data read per second: 360 reads * (506 + 6) bytes = ~180kb/sec  


### Step 2: Abstract Design ###

Let's start sketching out a top level overview of what our system will look like.  Our basic url shortening system will consist of two tiers: an application service layer and a data storage layer.  The application service layer is where our requests will be served and the data storage layer is where we will store our hash value to url mappings. You'll also want to give an overview of how the hashing will work.  

* Application Service Layer
  * Shortening service
    * Recieves a url and generates a hash value
    * Checks if hash value exists in data store
    * If it already exists, keep generating new hash values until an unused one is found  
    * If it doesn't exist, store a new hash value => url mapping
  * Redirection service
    * Recieves a hash value and does a url lookup
    * Returns a redirect to the long url

* Data Storage Layer
  * Acts like a big hash table 
  * Stores new hash value => url mappings
  * Retrieves a value given a key

* Hashing
  * Part of the application service layer
  * We'll use the MD5 hashing function, which will take the url and a salt
  * The hash value from the MD5 hashing function will need to be coverted to Base62
  * Then we'll use the first six characters of the Base62 value


### Step 3: Understanding Bottlenecks ###
  
To recap:
  * 400 req/sec
    * 360 read req/sec => 180kb/sec
    * 40 write req/sec => 20kb/sec
  * 3TB for all urls (over 5 years)
  * 36GB for all hash values (over 5 years)

The logic in the application service layer is very light weight, so handling 400 req/sec should not be that challenging to solve.

The bottlenecks are most likely going to be in the data storage layer.  The amount of data being read/written is small, but the number of reads and writes per second may cause issues.


### Step 4: Scalable Design ###

Coming Soon!

## Resources

**How to approach and work through system design questions**

[Hired In Tech - The System Design Process](https://www.hiredintech.com/classrooms/system-design/lesson/55)

**A summary of the Hired In Tech system design approach**

[System Design Cheatsheet](https://gist.github.com/banunatina/3959f128a8c7d20f79807fbccdf2e8bc)

**Scalability concepts that you'll need to know**

[Hired In Tech - Scalability Fundamentals](https://www.hiredintech.com/classrooms/system-design/lesson/60)
