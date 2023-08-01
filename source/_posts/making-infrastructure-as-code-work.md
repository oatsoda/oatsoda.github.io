---
title: Making "infrastructure as code", work
date: 2021-01-25 11:30:00
tags:
- Agile
- Azure DevOps
- Pipelines
- Process
---
_Updated 1 Aug 2023: General rewording and re-write of Develop locally section._

### Making Infrastructure as Code work

I believe there is a very simple way of thinking about Infrastructure as Code (IaC): 

> *It's how you ensure your deployed platform is saved in source control*.

Getting your deployment code in source control is just a case of committing it. So what else is there? Well, ask yourself the following question:

> ***How can you be sure that your Infrastructure as Code actually reflects your deployed platform?***

I will attempt to answer in this article.

![Infrastructure](colored-pencils.jpg "Source: https://pixabay.com/users/5598375-5598375/")

### Automate your deployments

Having your Infrastructure defined in deployment code is a great first step, but you should also make sure that it is then hooked up to automatically deploy from your CI/CD pipelines. This way there are no manual steps which could be subject to mistakes.  

Make sure that this is Continuous Deployment to your Test/Staging environments. In the same way that Automated Tests enforce the idea that all changes committed to product source _must work_, automated deployments each and every time changes are committed to source (both product code and deployment code) will help to enforce the idea that they are expected to work also. **Product code and Deployment code should both be treated as first-class citizens**.

### Work code-first

One of the major reasons that your deployed platform could drift away from what is defined in your IaC is that changes have been made to the platform but not committed to source. This could be changes made and then forgotten, or even just changes that have been "tried" in the deployed platform and are awaiting to be updated in code.

Either way, the two have drifted which is a bad thing.

This always reminds me of when I was a teenager working at my first job in a builders merchants. When we did a stock-take we were provided with an inventory of the what the stock system currently believe to be the count of each product. We would check or ammend in comparison to the actual amount on the shelf. We were always told "count from the shelf to the list". The idea being that psychologically if you read how many there were first, would you actually count the correct amount on the shelf, or just assume that while you counted it was the correct number? It's a similar concept with making changes on a deployed environment first (even if this is a Test/Staging environment) - if you made the change on the platform and then updated the deployment code afterwards, how do you know that your code works - perhaps the manual change was different from the code?

Where possible, make the changes in source ***first***. It avoids accidents of forgetting changes were made and also ensures that the deployment code ***actually works***.

### Develop locally

Following on from the concept of working code-first, you should also make sure you are developing locally and not against your deployed platform so that your test/staging environments are clean representations of your main source trunk IaC and not being constantly altered by developers working on core product features (especially by code that isn't even finished).

Use emulators or container images to give each developer their own isolated platform to develop against. I would also suggest that you don't name your CI/CD environment "Dev" or similar as, again, it suggests that it's part of development. Test or Staging are better names.

Developing locally may have some initial overhead with setup, but generally it is then faster for developers to work and test new work, with the added benefit they can do so offline. You can automate a lot of the setup with a set of scripts or using tools such as Microsoft's experimental [Project Tye](https://devblogs.microsoft.com/dotnet/introducing-project-tye/) (which hopefully they will soon announce how this will progress moving forward) and [Ngrok](https://ngrok.com/) (for incoming webhook calls).  Once you have these setup scripts (Be sure to commit those to source too - they are also your "local IaC") it is also then quicker for getting new team members up and running faster.

If specific elements of your employed environment don't have emulators or container images, then I would suggest you create instances of the element for each of your team members (again, automate this) rather than having shared instances (unless there is no data/state). If there's a cost implication with this, schedule in regular tear-downs to delete the resources when they're not being used.

#### Developing your actual IaC

Working on your actual IaC is slightly different as you aren't then working locally - you're working on code that needs to be tested against the destination platform before it gets committed to source. When doing this there is a sliding scale of discretion that is important to consider.

For example, if you were making a small change on one element of your infrastructure, but this element had _many_ dependencies then you need to decide whether the time to create that whole setup in order to then test the small alteration is worth spending. You may decide that the risk is minimal to run the proposed IaC change against your Test/Staging environment. By a "small" change I mean one that isn't vastly different to your current IaC, for example the setting of an additional property, and that change isn't incompatible with the rest of your main source trunk.

If you do this then you definitely want to be scheduling in tear-down (see below).

If you have a vast, complex platform which you regularly alter your IaC on a daily basis then instead you might want to add a specific environment for this and schedule in a nightly tear-down and morning creation.

### Bug investigation

When investigating issues on the deployed platform it can be tempting to make changes in an attempt to fix problems. I'd recommend trying to keep the discipline of working **code-first** and **developing locally** to solve these issues. This can be tricky when reproduction of the problem isn't easy - so you may need to use your discretion if the issue is critical. The danger here is that you may erode some benefit of the points above, so use explanations and training to ensure things are clear with your team.

### Schedule regular tear-downs of deployed platforms

Referring to the question right at the beginning: How can you be sure that your Infrastructure as Code actually reflects your deployed platform?  

Having CI/CD deploy your deployment code repeatedly is a great way of checking this, and predominantly you will be deploying to an already deployed environment. This is good. 99% of the time when you deploy to your Live production environment it will already exist, so you are testing the common scenario.

However, I also recommend occasional testing of a blank deployment for a couple of reasons:

1.  To check that your Test/Staging environment hasn't drifted.
2.  To satisfy yourself that in a Disaster Recovery scenario you can stand your platform back up quickly.

I'm talking about Test/Staging environments here: you probably want to also be running DR tests on your Live environment too, which are unlikely to be full tear-downs. This is out of the scope of this article.

If you do schedule in regular tear-downs, don't do it too often. Like I say, predominantly your deployments need to work on existing deployed platforms, so don't go too far the other way and have it deploy to clean environments every time as this could be masking problems which would occur when deploying to a deployed environment which is populated with more user data.

### Practice DevOps in your team

As a developer, I am coming at this from a coding point of view. DevOps is the idea that your Development and Operations are not separate - and Infrastructure as Code is the same. Whilst you could have separate Development and Operations and still utilise IaC, there's a good chance that your Development team are going to have more experience with the fact that source control is the *"single truth"* of your software, as opposed, potentially, to Operations who may not have used source control or the concepts of Continuous Integration and Delivery. Additionally, a true DevOps process works best because that shared responsibility across all members of the team should enforce everyone to treat Infrastructure as a first-class citizen of your product.

### Summary

Hopefully this article provides some ideas of how to approach IaC. Implementing these rules doesn't need to be draconian - hopefully you can trust your team and help them via coaching rather than locking down access.