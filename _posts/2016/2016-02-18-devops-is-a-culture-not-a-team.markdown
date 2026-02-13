---
layout: post
title: DevOps is a Culture, Not a Team
date: '2016-02-18 22:33:28'
tags:
- devops
- alm
---

_This post was originally posted on our [Northwest Cadence blog](http://blog.nwcadence.com/devops-culture-not-team/) – but I feel it’s a really important post, so I’m cross-posting it here!_

I recently worked at a customer that had created a DevOps team in addition to their Ops and Development teams. While I appreciated the attempt to improve their processes, inwardly I was horrified. Just as DevOps is not a product, I think it is bad practice to create a DevOps team. DevOps is a _culture_ and a _mindset_ that should pervade every member of your organization – beyond even developers and operations.

## What is DevOps?

So how do you define DevOps? [Donovan Brown](http://donovanbrown.com), a DevOps product manager at Microsoft, defines DevOps succinctly as follows: _DevOps is the union of people, process, and products to enable continuous delivery of value to end users_. Unfortunately, since the name is an amalgamation of _development_ and _operations_, most organizations get developers and ops to collaborate, and then boldly declare, “We do DevOps!”

## Wider than Dev and Ops

However, true DevOps is a culture that should involve everyone that is involved in _delivery of value to end users_. This means that business should understand their role in DevOps. Testers and Quality Assurance (QA) should understand their role in DevOps. Project Management Offices (PMOs), Human Resources (HR) and any other part of the organization that touches on the flow of value should be aware of their role in DevOps.

That’s why creating a DevOps team is a fundamentally bad decision. It distances people outside the “DevOps” team from being involved in the culture of DevOps. If there’s a DevOps team, and I’m not on it, why should I worry about DevOps? In the same manner, DevOps that is confined solely to dev and ops is indicative of the culture not pervading the organization. To fully benefit from DevOps, the entire organization needs to embrace the _mindset_ and _culture_ of DevOps.

## DevOps Values

What then is a DevOps culture? What values should be upheld by people as they improve their processes and utilize tools to aid in implementing DevOps practices? Here are a few:

1. Whatever we do should ultimately deliver value to the end users
2. This is absolutely key to good DevOps – everyone, from stakeholders to developers, to testers and ops should be thinking of how to _deliver value_. If you can’t ship it, it’s not delivered value – so fix it until you can deliver.
3. There’s no such thing as a DevOps Hero
4. DevOps is not the domain of a single individual or team. Everyone needs to buy in to the culture, and everyone needs to own it. And we need to build a culture of “team”, an [ubuntu](https://en.wikipedia.org/wiki/Ubuntu_(philosophy)) for value, within the entire organization.
5. If we touch it, we own it
6. If developers hand off their code to testers, then they ultimately assume “someone else” will check their code. If a developer is responsible for the after hours support calls, they’re more likely to ensure good quality. Of course enlisting the help of some testers will help that effort!
7. We should examine everything we do for efficiencies
8. Sometimes we need to step back and examine why we do certain things. Before we automated our deployments, we needed a change control board to wade through pages of “installation instructions”. Now we’ve automated deployments – so we do we still need the documentation or the checkpoint? We could go faster if we removed the “legacy” processes.
9. We should be allowed to experiment
10. Will automated deployment help us deliver value faster? Perhaps yes, perhaps no. We’ll never find out if we never have the permission (and time) to experiment. And we can learn from failed experiments too – so we should value experimentation

## Everyone has a responsibility in DevOps

DevOps is more than just developers and ops getting together and automating a dew things. While this is certainly a fast-track to better DevOps, the DevOps mindset has to widen to include other teams not traditionally associated with DevOps – like HR, PMOs and Testers.

### Human Resources (HR)

Human Resources should be looking for people who are _passionate about delivering value_. DevOps culture is built faster when people have passion and care about their end users. Don’t just inspect a candidates technical competency – get a feel for how much of a team player they are and how much they will take responsibility for what they do. Don’t hire people who just want to clock in and out and do as little as possible. Also, you may have to “trim the fat” – get rid of positions that are not _delivering value_.

A further boost for developing DevOps culture is the right working environment. Make your workplace a place people love – but make sure they don’t burn out too! Force them to go home and decompress with friends and family. If your teams are always working overtime, it’s an indication that something isn’t right. Find and improve the erroneous practice so that your team members can have a life – this will reinforce the passion and loyalty they have to delivering value.

### PMOs (Project Management Offices)

The PMO needs to rethink in many areas – especially in _utilization_. Most PMOs strive to make sure that every team member is running at 100% utilization. However, there are problems with this approach:

1. Humans don’t multitask
2. Humans don’t multitask – they switch quickly. However, unlike computers that can switch context perfectly to give the illusion of multitasking, humans have fallible memories. Switching costs, since our memories are not perfect. If there are no gaps in our schedules, we will inevitably run late since we don’t usually account for the cost of switching
3. No time for thinking, discussion and experimenting
4. If you’re at 100% utilization, you inevitably feel like you don’t have time to think. You can’t get involved in discussions with other team members about how to best solve a problem. And you can’t reflect on what is working and what is not. You certainly won’t have time to experiment. Over the long run, this will hamper delivery of value, since you won’t be innovating.
5. High utilization increases wait-time
6. The busier a resource is, the longer you have to wait to access it. A simple formula proves this – wait time = % busy / % idle. So if a resource is 50% utilized, wait time is 50/50 = 1 unit. If that same resource is 90% utilized, the wait time is 90/10 = 9 units. So you have to wait 9 times longer to access a resource that’s busy 90% of the time than when it’s busy 50% of the time. Long wait times means longer cycle times and lower throughput.

PMOs need to embrace the innovative nature of DevOps – and that means giving team members time in their schedules. And it means embracing uncertainty – don’t be afraid to trust the team.

### Testers

As Infrastructure as Code, Continuous Integration (CI) and Continuous Deployment (CD) speed the delivery time, testers need to jump in and start automating their testing efforts. In fact, just as I think that a DevOps team is a bad idea, I think that a Testing team is just as bad. Testers should be part of the development/operations team, not a separate entity. And traditional “manual” testers need to beef up on their automation skills, since manual testing becomes a bottleneck in the delivery pipeline. Remember, testers that “find bugs” are not thinking DevOps – testers that aim to automate their tests so that results are faster and more accurate are thinking about real quality improvement – and that means they’re thinking about _delivering value_ to the end users.

## Conclusion

DevOps is not a team or a product – it is a culture that needs to pervade everyone in the organization. Everyone – from HR to PMOs to Testers, not just developers and ops – needs to embrace DevOps values, making sure that value is being delivered continually to their end users.

