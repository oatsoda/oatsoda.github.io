---
title: Making "infrastructure as code", work
date: 2021-01-25 11:30:00
tags:
- Agile
- Azure DevOps
- Pipelines
- Process
---
### Making Infrastructure as Code work

I believe there is a very simple way of thinking about Infrastructure as Code: 

> *It's how you ensure your deployed platform is saved in source control*.

Getting your deployment code in source control is just a case of committing it.  So what else is there? Well, ask yourself the following question:

> ***How can you be sure that your Infrastructure as Code actually reflects your deployed platform?***

I will attempt to answer in this article.

![Infrastructure](colored-pencils.jpg "Source: https://pixabay.com/users/5598375-5598375/")

### Automate your deployments

Having your Infrastructure defined in deployment code is a great first step, but you should also make sure that it is then hooked up to automatically deploy from your CI/CD pipelines. This way there are no manual steps which could be subject to mistakes.  

Make sure that this is Continuous Deployment to your Test/Staging environments. In the same way that Automated Tests enforce the idea that all changes committed to product source _must work_, automated deployments each and every time changes are committed to deployment source (both product code and deployment code) will help to enforce the idea that they are expected to work also. **Product code and Deployment code should both be treated as first-class citizens**.

### Work code-first

This may sound strange, but one of the major reasons that your deployed platform could drift away from what is defined in your Infrastructure as code is that changes have been made to the platform but not committed to code. This could be changes made and then forgotten, or even just changes that have been "tried" in the deployed platform and are awaiting to be updated in code.

Either way, the two have drifted which is a bad thing.

This always reminds me of when I was a teenager working at my first job in a builders merchants. When we did a stock-take we were provided with a list which included how many items there were for each product for us to check or correct with the actual amount on the shelf. We were always told "count from the shelf to the list". That idea that psychologically if you read how many there were first, would you actually count the correct amount on the shelf, or just assume that as you counted it was the correct number?  It's a similar concept with making changes on a deployed environment first (even if this is a Test/Staging environment) - if you made the change on the platform and then updated the deployment code afterwards, how do you know that your code works - perhaps the manual change was different from the code?

Where possible, make the changes in source ***first***. It avoids accidents of forgetting changes were made and also ensures that the deployment code ***actually works***.

### Develop Locally

Following on from the concept of working code-first, you'll therefore want to make sure you are developing locally and not against your deployed platform.

Use emulators or container images to give each developer their own isolated platform to develop against. This enforces the idea that deployed platforms are representations of your main source trunk and not just a dump which can be played around with as part of development. I would also suggest that you don't name your CI/CD environment "Dev" or similar as, again, it suggests that it's part of development. Test or Staging are better names.

When it comes to actually working on your deployment code then the chances are there isn't an emulator or container image to represent this. This is where the temptation can be to use one of the deployed environments to develop against.  It's very tempting, and I'll admit something I have done quite a lot.  Ideally, you should provide developers with a separate account or place in an account that they can deploy their own items.  Use a naming convention which means they aren't interfering with other developers.  If there's a cost implication with this, schedule in regular tear-downs to delete the resources when they're not being used.

Creating and maintaining setup scripts for any developer machine pre-requisites can also be useful for getting new team members up and running faster.

### Bug Investigation

When investigating issues on the deployed platform, it can be tempting to make changes in an attempt to correct problems. I'd recommend trying to keep the discipline with working **code-first** and **developing locally** to solve these issues. This can be tricky when reproduction of the problem isn't easy - so you may need to use your discretion if the issue is critical. The danger here is that you may erode some benefit of the points above, so use explanations and training to ensure things are clear.

### Schedule regular tear-downs of deployed platforms

Referring to the question right at the beginning: How can you be sure that your Infrastructure as Code actually reflects your deployed platform?  

Having CI/CD deploy your deployment code repeatedly is a great way of checking this, and predominantly you will be deploying to an already deployed environment.  This is good. 99% of the time when you deploy to your live production environment it will already exist, so you are testing the common scenario.

However, I also recommend occasional testing of a blank deployment for a couple of reasons:

1.  To check that your Test/Staging environment hasn't drifted.
2.  To satisfy yourself that in a Disaster Recovery scenario you can stand your platform back up quickly.

I'm talking about Test/Staging environments here: you probably want to also be running DR tests on your live production too, which are unlikely to be full tear-downs. This is out of the scope of this article.

If you do schedule in regular tear-downs, don't do it too often. Like I say, predominantly your deployments need to work on existing deployed platforms, so don't go too far the other way and have it deploy to clean environments every time as this could be masking problems which would occur when deploying to a deployed environment which is populated with more user data.

### Practice DevOps in your team

As a developer, I am coming at this from a coding point of view. DevOps is the idea that your Development and Operations are not separate - and Infrastructure as Code is the same. Whilst you could have separate Development and Operations and still utilise Infrastructure as Code, there's a good chance that your Development team are going to have more experience with the fact that source control is the *"single truth"* of your software, as opposed, potentially, to Operations who may not have used source control or the concepts of Continuous Integration and Delivery. Additionally, a true DevOps process works best because that shared responsibility across all members of the team should enforce everyone to treat Infrastructure as a first-class citizen of your product.

### Summary

Hopefully that gives some ideas of how to approach Infrastructure as Code. Implementing these rules doesn't need to be draconian - hopefully you can trust your team and help them via coaching rather than locking down access.