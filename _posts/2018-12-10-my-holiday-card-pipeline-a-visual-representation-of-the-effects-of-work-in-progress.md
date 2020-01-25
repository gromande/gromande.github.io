---
layout: post
title: 'My Holiday Card Pipeline: A Visual Representation of the Effects of Work In
  Progress'
date: 2018-12-10 00:14:21.000000000 -06:00
password: ''
categories:
- DevOps
tags:
- devops
- lean
---
It's holiday time. Some people use this time of the year to reflect on the past. Others, prefer to look to the future and set goals for the new year. For me, well...for me it means sending holiday cards to friends and family.

This year I made a deal with my wife, she accepted to write the cards while I took care of sealing and stamping the envelopes. Halfway through this exercise, I remembered an example I read about in [The DevOps Handbook](https://itrevolution.com/book/the-devops-handbook/) that explained the drawbacks of having too much **work in progress (WIP)** in your work queue. The example goes like this...

<!--more-->

Let's assume that preparing a holiday card involves three discrete operations: insert the card into an envelope, seal the envelope, and put a stamp on it. To keep things simple, let's say that we need to send out three cards. What's the most effective way of completing this request?

One approach involves performing one operation on all three envelopes before moving to the next operation. I can insert all three cards into the different envelopes, then seal all three envelopes, and finally put a stamp on each one. This is called the "Large Batch" or "Mass Production" strategy typically seen in manufacturing. This is what a visual representation looks like:

![]({{ site.baseurl }}/assets/images/Envelopes-Flow-2.png)

As you can see, the time is takes to perform each operation is represented in time intervals (T1 to T9). If we assume that every operation takes 10 seconds, we can easily calculate the total time it takes to process the batch: _10 seconds/operation x 9 operations = 90 seconds_.

A different approach is to work on one envelope at a time. In other words, I can insert one card into an envelope, seal the envelope, and stamp it, before starting to work on the next one. This approach is called the "Small Batch" or "Single Piece Flow" strategy and it looks like this:

![]({{ site.baseurl }}/assets/images/Envelopes-Flow-1.png)

The time elapsed from start to finish is the same, 90 seconds. So which approach is better?

If we take a closer look, we can easily see that in the "Mass Production" strategy it took me 7 operations (i.e. 70 seconds) to finish the first envelope whereas in the "Small Batch" approach it only took me 30 seconds to produce the first envelope. In other words, the **throughput** (number of envelopes finished per second) is higher in the second approach. While the importance of throughput on a production system might not be obvious in this case (the finished envelopes will be sitting on the table until I walk to the post office and send them all at once), in other scenarios, throughput might be the difference between keeping your customers happy and going bankrupt. In software development for example, throughput is an indicator on how fast you can deliver value to your customers. In most cases, the higher the throughput, the happier the customer.

Something that we have not taken into consideration yet, is the effect that **context switching** has on the overall throughput. Context switching is also related to **multi-tasking**. As Dominica DeGrandis writes in his book [Making Work Visible](https://itrevolution.com/book/making-work-visible/): multitasking is an effective way to get less done. Let us add a new variable to the equation and assume that every time that we have to put an envelope down and pick up another one we incur a 2 second delay. In the "Small Batch" approach we switch between envelopes only twice, which increases the total elapsed time by 4 seconds. In the "Mass Production" approach however, we switch context 8 times! Which in turn, increases the elapsed time by 16 seconds.

What happens if we increase the batch size, say,&nbsp; from 3 to 30 envelopes? What about 1 million envelopes? Interestingly enough, in the "Small Batch" approach,&nbsp;other than the total elapsed time, the numbers don't change much. The **cycle time** (the amount of time it takes to finish one envelope) remains constant and the number of times we have to switch contexts grows at the same rate as the number of envelopes (_number of context switches = number of envelopes -1_). However, on the "Mass Production" approach, cycle time increases with the number of envelopes. It now takes even longer to deliver value to our customers. In addition, the number of context switches grows proportionally to the number of envelopes and the number of operations (number of context switches = [number of operations \* number of envelopes] -1). So the impact of context switching is also greater.

We can then say, that one interesting property of the "Small Batch" strategy is that the throughput is more predictable. I can confidently tell my wife that it'll take me ~30 seconds to finish one envelope independently of the number of cards. **Predictability** is a great quality to have in your value stream.

Things look even worse when all of a sudden we discover an error in the middle of the process. As soon as I was done sealing all three envelopes while using the "Large Batch" approach, my wife suddenly tells me that she forgot to sign the cards..."Oh boy, why couldn't you tell me earlier?" I said. "Now we have to open all the envelopes, sign the cards, and start all over again". "Well, we only have one envelope left so you'll have to go buy new ones" replied my wife. "Why didn't you work on a single envelope at a time? Your problem is too much WIP not me!". She was right of course. The problem would have been much easier and cheaper to solve if I had worked in smaller batches.

# Conclusion

My hope is that this simple visual example helped you better understand&nbsp;basic DevOps concepts such as WIP, Cycle Time, and Throughput just like it helped me.

To sum up, today we learned that reducing batch sizes can:

- Reduce work in progress (WIP) which in turn reduces Cycle Times and increases Throughput.
- Reduce the number of times we have to switch context which increases productivity and reduces stress.
- Increase the predictability of the value stream.
- Allow for faster detection of errors&nbsp;which&nbsp;reduces the cost of the fix.
