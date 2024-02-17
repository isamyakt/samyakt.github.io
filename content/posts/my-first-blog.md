---
title: How Data Structure perform operations?
description: Most people don’t know how data structure perform operations on data
date: "2023-08-04"
---

{{< figure src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Pb5PSgo4PxsZI2b96e7UCA.png" >}}

Let us start with the introduction to data structures. Data is an integral part of our application or programs. If I define a program, program is nothing but set of instructions which performs operations on data to get some results. So without data there is no need of instructions, no use of instructions, then the term ‘data’ we use it in many places like data structures, data-bases, data-warehouse, big-data. So from this, what actually is the data structure? before that if you have a little bit knowledge about all other things(data bases, data warehouse, big data) a that would be better.First I will start with data structures and we’ll also have the brief introduction to data-bases, data-warehouse and as well as big-data.

First let us start with data structures. What do you mean by data structure? Data structure can be defined as arrangement of collection of data items so that they can be utilized efficiently, operations on that data can be done efficiently.

So it’s all about the arrangement of data and the operations on the data that are efficient operation on the data. But the question is, where?Where does the operation takes place?

Obviously inside ‘Main Memory’ (RAM) during the execution of program. One thing I want to say that, if I use the word ‘main memory’ consider it as ‘RAM’ or vice versa.

I said this before but I’m repeating it again that without data structures there cannot be any application. Every application will have a set of instructions which will perform operations on data, so data is mandatory.

If data is mandatory, then where the data is kept during operation time? Inside the main memory(RAM). Then what about programs? I think you guessed right, programs are also inside the main memory. Data and Programs will be inside main memory during operation time.

What happens after completing the operation ? Where does data and Programs go? Obviously, you’re right inside SSD(Solid State Drive) / HDD (Hard Disk Drive).

Next question raises in my mind is that, during the execution of a program how the program will manage data inside the main memory and perform the operations. So I’ll take an example and explain you in detail how that program utilizes or manages data, how data and programs come to main memory and perform operations? Before that just stare this diagram

{{< figure src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*BYKRvYTbDDDjJ6krYOx8ZQ.png" >}}

Everybody know’s CPU, RAM, SSD and HDD. CPU is our microprocessor, this will execute our programs means during the process processor(CPU) will execute the instructions in our programs.

Main memory is just a temporary memory that is working memory. Which is also called as primary memory. Then What about SSD/HDD? They are permanent storage. So if we look into our PC, then we will have processor, RAM and Hard Disk or Solid State Drive. Or if we look into a mobile phone, then there will be a processor and there will be some RAM like 2GB, 4GB, 8GB and for storage 16GB, 32GB, 64GB, 128GB.. so on.

Then, where do we keep our programs? when we install any application or app in our PC or Moblie! Yes, you’re right apps will be stored in storage. You can see in diagram, where the ‘Program File’ and ‘Data File’ are stored in SSD or HDD. Suppose you have any pics or videos or any documents that all will be in your hard disk or SSD.

Now the Interesting part is here… For that, I’ll take one example of a commonly used application like ‘MS Word’, most of them used in there childhood. NotePad or any application used in these days like twitter, discord, instagram etc…. Sorry I changed my mind to take ‘Twitter’ app as example not ‘MS Word’. Assume that, Twitter as a program file and image or video as data file.

Now through this example, I’ll explain you where the data structure comes into picture and how Twitter may need data structure? Okay lets us see how our application program runs? If you want to run Twitter on your laptop or mobile, you are touching an icon or double clicking on an icon of a Twitter. Then let us see what happens next. As I said before stare this diagram 

{{< figure src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ZN9G2GPYyjRjgWEWvZasRw.png" >}}

Now you have to follow me on [Twitter](https://twitter.com/isamyakt).

Twitter program app will be brought into the main memory. You can see the lines in main memory, these are all the instruction of this Twitter program such that I’m showing line to represent the machine language code or instructions of Twitter Program.

So whenever you want to run an application. The application code or the program code has to be brought into main memory. Once the program code is brought into main memory, then CPU will start executing this Twitter application so you will see a window appearing on the screen in laptop or mobile and all the menu options everything comes up and you can start using Twitter.

Now, if you want to open a image file or document file or any data file, I will you image file in this analogy. Image file in your Twitter application, suppose this twitter program wants to access to the image in form of .png data file. Then this data has to be brought into main memory. Last diagram to stare

{{< figure src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*NQDh-Gw-tALaqF6y-jzKFg.png" >}}

Now from this, I can say that, a program has to be brought into the main memory for its execution, execution may be anything like uploading image in tweet or changing profile pic or may be anything, etc… During execution process, data file means image file has to brought into main memory for processing on the data. So, that these instructions can perform operations on the data mean image to change profile. Program can not directly process the data file keeping it on the storage, the data image has to be brought into the main memory.

Now, from this we can say that every application deals with some data, weather it is Twitter or Discord or Instagram or any application. If you are using Firefox or Safari or Chrome, then you are browsing any web page, then that page, where it is comming from Internet, It has to be in the main memory. So every application on your computer system or mobile deals with some data and data has to be inside main memory during execution.

Here is the Most Interesting Part: The next question a raises in my mind is that How you organize data inside the main memory? So that it can be easily used by any application program? So How you organize the data?

{{< figure src="https://miro.medium.com/v2/resize:fit:1400/format:webp/0*ahSsQj-iFMzigHfw" >}}

The arrangement or organizing of the data inside the main memory for efficient utilization by the application, that arrangement is called as Data Structure. So, data structures are formed in the main memory, during the execution time of a program. When the program runs it needs some data.

So the next question is how it will arrange the data in the main memory for performing its operations?

You may be knowing different data structures like array or linked list or hash or graph etc… Whatever the data structure is suitable for application can use that particular data structure here for arranging its data.

So what kind of data? Data can be a text data, or media data like I used as example of Images, or videos. Lot of contents maybe there in the form of data. So all those contents, how they are organized in the main memory. So that we have to design some data structure so that the application can use it perfectly or more efficiently and application should work faster or process faster over that data. That's it!

I have given you the idea, what is Data Structure and how it will be used in your computer or laptop and much more. I hope you enjoyed reading.