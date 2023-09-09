# streaming-05-smart-smoker

## Producer Requirements

PACKAGES:

Pika
Sys
Webbrowser
CSV
Time
Logging


FILES:

smoker-temps.csv

## Consumer Requirements

PACKAGES:

Pika
Sys
Time
Logging
Collections
Datetime

## Producer Notes

Overall, this script is very similar to v2_emitter_of_tasks.py in Module 4, as well as my own version of that, v3_emitter_of_tasks.py.

However, I did change some names of things. For example, the original function that opens the RabbitMQ admin site has been renamed to "admin()" for simplicity, per the instructions found here: https://nwmissouri.instructure.com/courses/54849/pages/module-5-dot-2-guided-producer-implementation?wrap=1 

In my module 4 assignment, I had a function to send a message to a queue, and one to open and read a csv. The same general principle applies here. However, most of the heavy lifting is being done by a function called "main()", which was named by following the directions in the link above. The main() function deletes existing queues before starting anything else, declares new queues, reads values from each column and row in the specified CSV file and assigns those values to variables, accesses a custom function to send messages to queues based on which column is being read from the csv, and finally closes itself out once finished. There is a lot going on here, but again, most of this is standard from our other assignment, save for some of the reading of the CSV file. However, we have seen the "with open()" convention in other classes, so this is nothing new. The assignment instruction specifically described issues with the values in the CSV file being floats, while the messages needed to be strings. To mitigate this, I read those values as floats and then added the line "if row [1] else None " to handle any missing values so as not to cause problems with the floats. Per the instructions, I also used the time.sleep() function to send only one record every 30 seconds. This falls with in the overall "with open()" loop to ensure that each message (row in the CSV file) takes 30 seconds to send.

Before the main function, I defined a custom function called send_message(). This takes as arguments the channel, the queue name, the timestamp, and the temperature, all of which are defined in the main() function. Each queue name is associated with the different temperature types (columns in the CSV) that the smoker reads, so each temperature goes to it's respective queue. Other than comments, there are only 2 lines of code here that we have seen before, channel.basic_publish() and logging.info(). These tell the program what messages should go where and what should be displayed to the user.

To interrupt and stop the program, type CTRL+C.

## Consumer Notes

Like the producer script, this one is fairly similar to the source consumer script we used in Module 4, v2_listening_worker.py and my version of it. There are several more in depth changes added to handle multiple queues, callback functions, the conditions associated with those, and various other error and oddity handling. The overall goal is to be alerted when a critical temperature of the smoker or food is reached.

After the imports and some logging setup, a few variables are defined regarding the number of items that should be held in the deques (based on the 30 second iterative sending of messages) and the temperature threshhold values.

Then, a new function, process_temperature() is defined, with the overall goal of decoding the message from the producer and logging events, whether that be temperature recordings or temperature spikes. Very generally, the function works by unpacking the message, performing some conversions, recording temperatures and times from the deques, and logging messages if events occur and temperature spike conditions are met. This is a simplified explanation. More detailed information can be found within the comments of the code itself.

Another new function, get_time_window(), was added to figure out what the critical time window is that a temperature spike should occur within to trigger an alert. This function is rather simple, checking a temperature name (either "Smoker", "Food A", or "Food B") to decide what the time window value should be.

Then, three new callback functions are added, one for the smoker, one for Food A, and one for Food B. These are also rather simple, sending work to the process_temperature() function based on which temperature is relevant.

The main() function is almost exactly the same as the original main() function in module 4. However, a list of tuples containing queue names and their associated callback functions created and looped over, telling the program which queue should be accessed based on which callback function is needed.

When running this process in a terminal, interrupt and stop the program with CTRL+C.

## Screenshots for Producer

![Script running in the terminal](./Terminal.JPG)

![RabbitMQ Admin Site as the producer script is running](./AdminSite.JPG)

## Screenshots for Consumer

![Terminal showing smoker events](./SmokerTempChangeAlerts.JPG)

![Terminal showing smoker and Food A and Food B events](./SmokerAndFoodTempsAndAlerts.JPG)

![RabbitMQ Admin Site as the consumer script is running](./RabbitMQConsole.JPG)