---
layout: post
title: A Clever Regression Testing Approach
---
My new client and project has been alive quite a time and has involved many people and consultants going in and out. It also has a large code base and typically, these combinations entail code being written in different styles making it harder to read and maintain. However, involving many people can also result in rise of new and smart solution to common problems IF the work environment facilitates innovation. This is an entire subject in itself, which I will not dwell into here, but my client is one that actively supports innovation from its employees. I will describe one smart approach that has been developed in the project that I learned recently for regression testing. This testing approach allows us to regression test most important parts of our code base and involves a lot of test data, but without requiring too much manual verification

## The need for cheaper regression tests
The client consists of around 8-9 software development teams, each responsible for their own business module. The module my team is working on consists mainly of a large library which contains complex business logic. Other teams are allowed to commit code into our components if they need to do minor changes as a result of changes in their own module. To allow such behavior in a complex and large code base requires thorough testing, and the normal unit and integration tests are not enough. Regression testing is essential to ensure quality, but can be expensive and boring if done manually before each release. We need a clever and cheaper way of regression testing which is au pair with manual test!

## System Background
The system we are working on is a caseworker application, where a lot of rules, validations and calculations are performed in a particular order. Our module handles an important and complex part of the case processing. The changes in each development cycle (agile, 3 week sprints) are mostly in the library because it contains the case processing logic. However, we also have some batches that use the library for automating business processes, which are more stable with regards to changes.

## The cheap and clever regression test
Our goal with the regression test is to ensure that there are no unintended changes introduced during the development cycle. If we can perform case processing on only our module and verify that the results are as expected, we have reached our goal. Since we support automated processing through batches, we are able to trigger the most important case processing logic in correct order. The challenge lies in verification. A manual verification would involve domain experts/case workers to go through the result of the automated processing and verify that it is as expected. This procedure is feasible for a small number of customers because it requires a lot of time. As an alternative, we could write application that goes through the result and performs some assertions, but then we have to maintain it and keep it up to date.

Our approach is a different one, which is to compare the automated processing results with a previous version; we first run the automated processing on a production version of the library. Then we run it on the development version _with the same initial data set_ as the previous run. Finally we compare the data sets to see what have changes and create reports of the differences. Because the initial data sets are the same, the differences should either be expected due to new functionality or are a result of bugs.

The method is illustrated in the figure:
![Regression testing method][regression_testing]

The main benefit of this method is that since we operate on deltas or difference from the previous versions we are freed from creating and maintaining assertions. This approach can thus be used continuously without requiring updates after each development cycle. On the other hand, we must involve at least one domain expert/case worker to verify what the differences mean. The verifier can focus solely on the actual differences, instead of going through all data, which requires much less work. In addition, we aggregate the type of differences to create a summary, for example "340 customers had a change in table A column 2". This summary makes the verification process even easier. Overall, we get a great thorough regression test cheaply!

## Performance, automation and CI
To make this approach even greater we should make it automated, which we are in the process of doing. A major obstacle for us is that the complete test takes a lot of time because the automated process batch requires some setup before it is run and the run takes a lot of time. In addition the comparison part uses at least same amount time. Usually we spend around 3 hours on setup and initiation, 10 hours on processing including comparison, and 1-2 days on analysis of the results. We are looking into making the setup and initiation automatic so that the regression test can be run in weekends. We can decrease the time the processing takes by adjusting the data it processes, but this will also affect the quality of the test.

The ultimate goal would be to involve the regression test in our continuous integration cycle. On each build, the test would run with a very small subset of data, while in weekends we would run with normal data set. The problem is that we cannot fully automate the final analysis part. However, we can add some general assertions that operate on the difference summaries - by for example making limits on the number of differences. Instead of assertions, we can do it even simpler by plotting the number of differences per type in a graph. For each test the graph is updated making it clear if a commit resulted in an increase in difference.

## To summarize...
So, to be able to use the approach described you should have:
* A batch/script/program that is able to call most part of business functionality just as a user would through its user interface, with correct steps in correct order. In my experience, most systems where users perform tasks in steps, like caseworker application, often already have an automated processing of steps.
* Ability to access a lot of test data for regression testing, and loading it to multiple databases.
* A batch/script/program that performs comparison of the two datasets and produces reports. This part should be cleverly designed so that it does not become a bottleneck, for example by supporting parallel processing and doing all I/O before starting comparison.
* Access to verifiers that can assess the results of the comparison.

This approach gives us high value low cost regression test across a large code base
* the test involves large parts of our code using real and lots of test data
* the test code requires little maintenance because of its nature
* we need to involve few verifiers

There may be other methods out there, which solve things in a different way, and I would love to hear about them! Please leave a comment!

 [regression_testing]: {{site.url}}/assets/regression_testing.png "Regression test method"

