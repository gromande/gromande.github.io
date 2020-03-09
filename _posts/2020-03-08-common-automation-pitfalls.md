---
layout: post
title:  "Automate All the Things? Common Automation Pitfalls in IT"
date:   2020-03-08
tags: automation
---
Automation is not a new concept by any means. It has been around since the industrial revolution and it is the main reason why we have become so efficient at producing "things" at an ever faster pace. It was first applied to the production of physical "things" such as cars and electronic devices. Today, IT and Software Engineering teams around the world rely on automation to keep up with increasing demands to reduce costs and minimize time to market.

It is undeniable that automation has many great benefits: higher throughput, scalability, stability, and consistency among many others. But when used recklessly or applied to the wrong context, it can have unintended consequences that aggravate the problems it was supposed to solve in the first place. In this post, I want to highlight some of the most  common automation pitfalls I have seen throughout my career in IT and Information Security.

## Knowing when to automate something and why
I usually find it very useful to lead with the following question: What problem are your trying to solve? If you are not able to define the problem statement, chances are you won't succeed in solving it. In trying to answer this question, you might realize that you are just dealing with a symptom rather than the problem itself.

Once you've been able to pin down the problem, it is equally important to understand why automation should be part of the solution. I recommend IT teams to use automation not because is cool or fun to implement, but because it makes sense for the given problem context.

For example, I know of several IT Operations teams that keep a count of how many times they receive a request to do something manually, like granting access to applications and systems, or running reports for the other departments. If number of requests crosses a threshold in a short period of time, they automate the process.

A mental formula that helps me frame the conversation is as follows:

*Return on Investment (ROI) = Time Saved - Implementation Time*

**If the time savings expected from automation is less than the time it will take to automate a process, the process is probably not worth automating**. You might also want to take into consideration the time you will have to invest maintaining and updating the automation.

Early on in my career, I was helping a financial services company with the installion of a pretty large 3rd party authentication and authorization system. For security reasons, they did not give me access to the environment so I decided to create a script to automate the installation and configuration process. I thought that the investment up front would pay off eventually, since I wouldn't have to depend on somebody else doing the installation manually.

My thought process turned out to be flawed for several reasons:
- I ended up spending a lot more time developing the scripts than I had originally anticipated. Humans usually underestimate how long things will take.
- I wasted a lot of time troubleshooting issues and fixing bugs in my script.
- And most importantly, after all the time and effort I put into it, the script was used just one time. As soon as the system was up an running they threw the script away and never used it again.

If I had spent a couple minutes thinking about the ROI I would've realized that I was better off spending my time walking somebody else through the installation process in person.

## You can automate anything...even a mess
I cringe whenever I hear people say "Automate all the things!". It is true that with the latest advancements in technology we can automate almost anything. But should you?

Manual tasks are a main source of frustration among IT employees, so it makes sense that people want to delegate those tasks to a machine. However, I tell my clients to start by defining and documenting the underlying process. Have a whiteboarding session with the stakeholders, document the process on paper, create flow diagrams, find ways to optimize it. **Make sure the process has been clearly defined before trying to automate it**. I've seen many companies build bad automation on top of bad business processes.

## The amplification factor
You have to be really careful not to introduce issues into the process since automation can dramatically amplify the damage.

This is a common debate in the DevSecOps community. One of the goals of the DevOps movement was to reduce the time it takes to deliver value to customers. A lot of people mistakenly equate this with having more frequent deployments. Even though speed is clearly a competitive advantage, speed without quality can be detrimental in the long run. Automation allows us to develop and deploy new features faster, but it also opens the door for bugs and vulnerabilities to be introduced at a faster rate if the right controls are not in place.

**Use development standards like version control, configuration as code, and automated testing to to increase quality of the automation without hindering speed.**

## Don't forget about the people
Up to this point, we've only discussed the technology and process aspects of automation. But what about the people aspect? Who will benefit from the automation? And who will operate and maintain it moving forward?

If your are building automation to increase the productivity of your team, you probably have a good idea of how the process should behave. However, if you are building automation for somebody else, you should talk to your "customers". **Whether your customers are internal employees, external customers, or partners, you need to talk to them and gather requirements. Don't waste your time developing features that are based on your own assumptions without proving them first**.

You also need to ask yourself who will be responsible for maintaining the code. We live in an ever changing world and chances are that your code will soon become obsolete. I've been part of many IT migrations and modernization efforts for companies that are trying to move away from legacy systems and applications that were built in-house many years ago. Oftentimes, the people that built and understood the system are no longer with the company. **Keep a long-term perspective when designing an automated processes and develop the system in a way that other people can understand what it does and maintain it**. Somebody will thank you later.
