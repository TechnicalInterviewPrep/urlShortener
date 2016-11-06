# URL Shortener

![bitly image](https://cdn2.hubspot.net/hub/431273/file-3019062849-jpg/blog-files/bitly-logo.jpg)

## System Design Challenge

Design a URL shortening service like [bitly.com](http://www.bitly.com).

If you're new to system design problems or are not sure where to start, check out the Resources section below.


## The Solution

### Step 1: Constraints and Use Cases

#### Use Cases ####

The first thing that you want to do is ask the interviewer questions in order to clarify the use cases for your system.  For our Bitly clone we're going to have two main use cases.

1. URL Shortening - recieve a url from the user and return a short url
2. Redirection - recieve a request to a short url and redirect to the original url

Other potential use cases that you could ask about include:
  * Analytics
  * Automatic link expiration
  * Manual link removal
  * UI vs. API

#### Constraints ####

The next thing that you want to do is clarify the constraints.  These are the factors that you will need to take into consideration as you design your system.  The constraints that we'll start with are load and data.

##### Load #####
You need to figure out how much traffic your system will be required to handle.  For this example, a good breadown of monthly traffic would be:
  * Total number of requests per month
  * Number of requests that are for url shortening (writes)
  * Number of requests that are for redirection (reads)

*You: How many requests per month does the system need to be able to handle?*  
*Interviewer: Assume that it's not going to be in the top 3 url shortening services, but it will be in the top 10.*

We know that Twitter users generate 500 million tweets/day => 15B tweets/month  
Let's assume that 5-10% of tweets use a shortened url  
That would mean the total number of new urls being shortened each month is roughly 1.5B  
Let's assume that 80% of requests go to the top 3 link shortener sites => 1.2B  
The remaining 20% of requests go to all of the other link shortern sites => 300M   
If we're one of the top 10 link shortener sites, but not one of the top 3, a conservative estimate for the number of links that we shorten a month would be 100M

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
  * How much data will be passing through our system during peak loads?
    * Reads
    * Writes

Let's assume that we'll be desiging the system to be able to store 5 years worth of urls  
Total urls: 5 years * 12 months/year * 100M urls/month = 6B urls  
let's assume that the average length of a url is 500 characters  
1 char = 1 byte  
500 bytes per URL => 3TB for all urls  

We need 6B unique hash values   
For our hash values, we will use Base62 (numbers, uppercase letters, and lowercase letters)  
5 characters long => 62^5 = ~1B unique combinations  
6 characters long => 62^6 = ~57B unique combinations   
Each hash value will need to be 6 characters long   
6 bytes per hash value => 36GB for all hash values  

New data written per second: 40 writes * (500 + 6) bytes = ~20kb/sec 
Data read per second: 360 reads * (506 + 6) bytes = ~180kb/sec  


### Step 2 - Abstract Design ###

## Resources

**How to approach and work through system design questions**

[Hired In Tech - The System Design Process](https://www.hiredintech.com/classrooms/system-design/lesson/55)

**A summary of the Hired In Tech system design approach**

[System Design Cheatsheet](https://gist.github.com/banunatina/3959f128a8c7d20f79807fbccdf2e8bc)

**Scalability concepts that you'll need to know**

[Hired In Tech - Scalability Fundamentals](https://www.hiredintech.com/classrooms/system-design/lesson/60)
