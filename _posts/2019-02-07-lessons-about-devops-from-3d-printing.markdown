---
layout: post
title: Lessons about DevOps from 3D Printing
date: '2019-02-07 01:11:20'
tags:
- devops
---

1. TOC
{:toc}

It's no surprise that I'm passionate about DevOps. I think that has to do with my personality - my top five [StrengthsFinder](https://www.gallupstrengthscenter.com/home/en-us/strengthsfinder) strengths are Strategic, Ideation, Learner, Activator, Achiever. I love the combination of people and tools that DevOps brings together. Being deeply technical and also fascinated by how people interact means I'm built for DevOps consulting. Add to that my love of learning, and I'm in my perfect job, since if there's one thing I've learned about DevOps - it's that I'll never learn everything there is to learn about it!

<!--kg-card-begin: html--> [![IMG_20190120_001801](/assets/images/files/c6ecad64-eefb-40d6-819e-327fc336a254.jpg "IMG\_20190120\_001801")](/assets/images/files/8feab4fd-e6bb-48e4-a8c6-ea80d50779ba.jpg)<!--kg-card-end: html-->

I recently got a 3D printer (an [Ender3](https://www.youtube.com/watch?v=odDZMYr1di8)) for my birthday - I was actually looking for some Raspberry Pi projects to do with my kids when I saw a post about a guy who created some programmable LED Christmas lights. He'd printed himself a case for his Raspberry Pi and I wondered, "How much does a 3D printer go for nowadays?" I was pleasantly surprised to learn that you can get a pretty decent printer for around $200. So I got one for my birthday - and it's been a ton of fun learning how to model, then slice models, then tweak the printer to make it bend to my will. But watching an abstract model turn into physical reality before your eyes is fantastic!

While I was learning how to 3D print, I realized there are a lot of parallels between 3D printing and DevOps. Some may accuse me of "when you have a hammer, everything looks like a nail" and I suspect they're partly right. But that doesn't mean you can't learn about one discipline from studying another! Remember, Lean and Agile have roots in auto manufacturing!

<!--kg-card-begin: html--> [![IMG_20190203_205203_Bokeh](/assets/images/files/b6a00ae0-e341-41d4-b811-b91a25fbec20.jpg "IMG\_20190203\_205203\_Bokeh")](/assets/images/files/0e51842d-b2b4-42b6-9274-0fa0b369361f.jpg)<!--kg-card-end: html-->

So here are some thoughts about DevOps that I got from 3D Printing.

### You Have To Just Jump In At Some Stage

I ordered my printer just before Christmas (my birthday is in January) but I had about 3 weeks between ordering and receiving my printer. I watched a ton of videos, read as much as I could, learned a couple of modeling and slicing programs - but no matter how much I read, I had to just start printing! There's something about learning while you go that's fun (and sometimes frightening) about both 3D printing and DevOps. Don't get into "analysis paralysis" - start somewhere and you'll be surprised how quickly you can move. Having a partner who can help you decide on some "low-lying fruit" will help you start faster - but don't be afraid to start somewhere. Also, no matter how much theoretical knowledge you have, you're going to have to start implementing and then adjust along the way.

### Understanding Fundamentals Improves Your Success

While it was frustrating to wait so long for my printer to arrive, I am ultimately grateful because I learned a lot of fundamentals. Before ordering the printer I had no idea what [PLA filament](https://en.wikipedia.org/wiki/Polylactic_acid) was or what [slicers](https://en.wikipedia.org/wiki/Slicer_(3D_printing)) were. But learning the fundamentals of how printers actually work has helped me troubleshoot and improved my success.

Learning about DevOps can help you on your journey - some teams implement automation and call it DevOps. This shows that they don't fully understand what DevOps is about - and understanding the higher-level goals and history of DevOps can help you on your journey. Don't just jump onto the buzz words - understand _why_ they make a difference. Understanding fundamentals will help you improve your success.

### Small Changes Can Have Radical Impact

Because [Fused Deposition Modeling](https://en.wikipedia.org/wiki/Fused_filament_fabrication) (FDM) printing is additive, the 1st layer is critical. This layer needs to bond correctly to the build bed, otherwise the print is doomed. Adjusting the bed to make it level and the correct height from the nozzle is a fiddly task - and small changes can make a huge difference.

In DevOps, sometimes small changes make a big difference. Be mindful of how you make changes in your teams, processes or tools. Making too many changes at once will prevent you from determining which changes are working and which are not. Also small changes let your team get some wins and build momentum.

### Just When You Think Everything Is Perfect, Something Fails

One of the challenges with printing is extrusion - the amount of plastic that is fed into the hot-end as the printer works. Too little and you get holes and missing layers, too much and you get blobs and stringing. The printer firmware has a multiplier for the extruder - if you program it to extrude 100mm of filament, it should extrude 100mm of filament! However, the stepper motor isn't perfectly calibrated, so you have to tweak the multiplier to get the correct extrusion. I had gone through the process of setting the extruder multiplier and was happy with the prints I was getting. I wanted to install some upgrades and wanted to print a baseline print for comparison - but the baseline print was terrible! There was clear under-extrusion - which I wasn't expecting since I hadn't touched the extruder settings. Eventually I had to recalibrate the extruder multiplier again.

In DevOps, you never "arrive". DevOps is a journey, and things can sometimes just blow up when you least expect. Remember that DevOps is more than just tooling and automation - people are a critical component of DevOps. And people change, new people come in or leave - and these changes can affect your culture - and therefore your DevOps. Keeping your eyes open, ensuring that you're following the vision and making sure everyone is still with you is key to success.

### Fast Feedback Is Critical
<!--kg-card-begin: html-->[![IMG_20190123_233331](/assets/images/files/1201059e-78b5-4ecd-8c5a-172c21c2a3cc.jpg "IMG\_20190123\_233331")](/assets/images/files/879660a1-da7c-42e5-b2d3-42085ba893a9.jpg)<!--kg-card-end: html-->

Some prints can take a while - the Yoda print I made for my son took just over 7 hours. I watched closely (especially in the beginning) to make sure the first couple layers worked correctly - fortunately I got a good couple layers early on and the print turned out great. I have also done some other prints where the first couple layers didn't bond to the build surface correctly, and I aborted before wasting time (and filament). Getting feedback quickly was critical - fortunately I got to see the layers as they ran, so I got immediate feedback.

Getting feedback quickly is one of the primary goals of DevOps - reducing cycle times ensures that you can iterate rapidly and adjust rapidly. You may have heard the expression "Fail fast". Rather get feedback after 2 weeks and adjust than go off for 3 months building the wrong thing. Whatever you do and however far you are on the DevOps journey, make sure that you get rapid feedback - both for the software you're building and for your DevOps processes (how you're building) so that you can adjust quickly and often.

### You Can Use a Printer To Improve a Printer

It's almost a rite of passage - when you get your printer, your first dozen prints are upgrades for your printer! Every print you make for your printer improves your printer so that you keep getting better prints.

In DevOps, you can apply principles for good software delivery to the process itself. How about "evidence gathered in production"? There's no place like production, so that's where you want to get usage and performance metrics from. Similarly, you want to measure your team in their native habitat to see how they're doing. Reducing cycle times and getting feedback fast improves your software, but are you applying the same principles to your processes? Try something, evaluate, and abort quickly if it's not working.

At some stage, however, you need to print something other than parts for your printer. If all I did was print parts, then the parts become pointless since I bought the printer to enable me to print prototypes or art or whatever, not just printer parts. Don't navel gaze too much into your DevOps processes and remember that the ultimate goal is to deliver value to your customers!

### Sometimes You Have To Cut Your Losses And Try Again

I love board games - and we have a good collection of them. I wanted to print some inserts for sorting the cards and components to [Dead of Winter](https://www.plaidhatgames.com/games/dead-of-winter) - and the prints kept failing. I got a bit frustrated and stopped printing (for now) until I've got some confidence back.

In DevOps, you can try things and if they fail, you may have to cut your losses and start again. Perhaps you need to try again - perhaps you need to try something different. Don't give up because something didn't work like you expected it to. Regroup, recoup and try again!

### Your Printer Is Unique

I'd seen a few experts recommend [TL smoothers](https://www.youtube.com/watch?v=hUL-g3MUvJY) to improve prints. The stepper motors on a printer are driven by voltage differentials, and the TL smoother is a set of diodes that prevent a voltage "flutter" when the voltage dips towards 0 - which smooths the movement of the stepper motor. So I decided I want to get some. I got a baseline print, installed the smoothers and repeated the print. Absolutely no difference. To be fair, the smoothers only cost $12 so it's not a big deal. It turns out that my Ender3 probably has better components than some earlier models, so the smoothers weren't necessary, even though experts had recommended them.

With DevOps, you're far better off being pragmatic about how and when to apply changes in your processes, tools and people. Don't just implement blindly - understand how DevOps practices will affect your team and how best to implement them for your team. Just because another organization or team was successful with some practice, doesn't mean you have to do it in the same way (or at all). Each team, environment and organization is different, so your DevOps could look different from that in other orgs. As Dori says, "Just keep swimming!"

## Conclusion

There's a ton to learn about DevOps from 3D printing. As with anything, we're all on a journey. But sometimes we have to remember to step back and remember how far we've come - and why we're on the journey in the first place! Hopefully this reflection is positive for you.

Happy printing!

