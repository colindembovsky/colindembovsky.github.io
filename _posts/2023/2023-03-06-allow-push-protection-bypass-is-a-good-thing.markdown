---
layout: post
title: 'Allowing Bypass of Secret Scanning Push Detections is a Good Thing'
date: '2023-03-06 01:22:01'
image: /assets/images/2023/03/bypass/secret.jpg
description: >
  Secret Scanning Push Protection allows you to block pushes that contain secrets. These blocks can by bypassed, which may be surprising. However, allowing bypasses is actually a good thing!
tags:
- security
---

1. TOC
{:toc}

> Image by [Tim Hüfner](https://unsplash.com/@huefnerdesign?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/target?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

[GitHub Advanced Security](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security) includes [secret scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning). While there are other secret scanning solutions in the market such as [TruffleHog](https://trufflesecurity.com/trufflehog/), no SaaS solution can offer _push protection_.

## Secret Scanning Locations

Secret Scanning could be implemented in 3 locations:
1. The local developer environment - either in the IDE or in the CLI
1. In a build after commits are pushed
1. At the time of the push

Let's examine the pros and cons of each of these approaches.

### Local Environment

Performing secret detection in the local environment only works as long as developers remember to run the tool. And if their favorite IDE doesn't support the tool, it's unlikely that they'll run it. Furthermore, even if developers remembered to run these detections every time before they pushed, how would organizations manage custom secret patterns or other configurations? Centralized configuration is essential for managing security at scale - so organizations can't just think of the _scanning_, they have to think about how they would manage custom configurations too.

### Post-push in a build

If the local environment is too heterogenous and relies too much on the developer, then surely adding a scanning tool in the build makes sense. That way, teams can guarantee that the scan is being performed and could manage configuration using [reusable workflows](https://docs.github.com/en/enterprise-cloud@latest/actions/using-workflows/reusing-workflows).

However, this is _too late_ in the life cycle - the secret has to be the repo for the build to perform the scan. While this option adds more consistency, it cannot prevent the secret from getting to the repo in the first place.

### At push time

The best place to scan for secrets is at the moment of the push. Teams could do this using [pre-receive hooks](https://docs.github.com/en/enterprise-server@3.8/admin/policies/enforcing-policy-with-pre-receive-hooks/managing-pre-receive-hooks-on-the-github-enterprise-server-appliance) on GitHub Enterprise Server. This would allow teams to run some validation on the push and allow or block it - say, if it contained a secret. Unfortunately, GitHub Enterprise Cloud does not support pre-receive hooks (yet).

However, GitHub Advanced Security does include the option to enable [push protection](https://docs.github.com/en/enterprise-cloud@latest/code-security/secret-scanning/protecting-pushes-with-secret-scanning). This prevents pushes if secrets are detected.

This push protection feature is unique in the market for several reasons. Some tools have _some_ of the features listed below, but only secret scanning push protection in GitHub Advanced Security has all of the following:
1. It is embedded into the repos and can be enabled instantly at enterprise, org or repo level
1. It does not require build customization or IDE plugins or anything else - it simply works
1. It allows admins to create custom patterns that are managed centrally
1. It allows admins to perform dry-runs of their custom patterns so that they can refine them before they roll them out, preventing noise and loss of developer trust
1. Alerts trigger webhooks for additional automation and alerts are also visible in the audit log

However, it is important to note that push protections _can be bypassed_. But why? Wouldn't you want to hard-block any detected secrets?

## Allowing bypassing is a good idea

This seems counter-intuitive. However, let's think about why this actually makes more sense that preventing bypasses.

### False positives

There are rare cases when secret scanning will detect what it thinks is a secret - but it's not in fact a secret. In these cases, a bypass is crucial since you need to get the code into the repo. This becomes even more critical as admins roll out custom patterns, especially for "generic" secrets (like database connection strings) which have no governing pattern (unlike tokens which tend to have much more predictable patterns). The less predictable a pattern is, the more noisy (more false positives) it is going to generate.

### Maintaining trust with developers

Whenever there is a gate, control or roadblock in the development life cycle, there must be some real value in the gate. Too many controls are vestiges of old processes or created by people who are no longer at the company, but are not challenged. This leads to friction and causes developers to lose trust in the security teams (or IT teams) and vice versa. Developers will also start to lose trust in the platform.

Totally preventing bypasses of push detections is effectively a statement that _you do not trust your developers_. Most developers are not malicious and secrets in pushes will most commonly be mistakes: a dev is testing and puts a credential to a test platform or database in their configuration file, only to forget to remove it before pushing. In this case, the push protection helps remind the dev that they have a secret that should not be committed to the repo. So allowing bypasses for false positives while preventing _accidental_ leaks is a good combination.

### Workarounds

Let's imagine a scenario where push protections can never be bypassed. Developers who experience false positives will be frustrated since they have no way around the incorrect detection. This may lead them to become creative and find workarounds.

For example, developers could simply `base64 encode` the secret. This results in a high-entropy string. High entropy strings could be added to push detection, but by nature will produce a lot of noise (lots of false positives). So in all likelihood, these base64 encoded strings would end up being pushed to the repo. This is a leak, since you can simply `base64 decode` the string to get to the secret.

Or a developer may take a credential and split it in half, and simply concatenate the halves at run time. Again, an extremely difficult scenario to detect, but easy for a human to exfiltrate.

In short, workarounds make detection harder, and so increase risk.

> Note: I have heard stories from customers who have created their own secret scanning tools that cannot by bypassed. The results were disastrous, and the tool is either turned off or bypasses have been allowed.

## Effective management of bypasses

This doesn't mean that allowing bypasses is insecure! With some simple steps, organizations can implement effective controls for bypasses, allowing them to retain customer trust as well as prevent secrets from leaking.

There are two primary methods to track bypasses of push protections:
1. The `secret_scanning_alert` [webhook](https://docs.github.com/webhooks-and-events/webhooks/webhook-events-and-payloads#secret_scanning_alert) which is fired every time a protection is bypassed (the `push_protection_bypassed` property is set to `true`)
1. The `secret_scanning_push_protection` category of [audit logs](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/audit-log-events-for-your-enterprise#secret_scanning_push_protection-category-actions)

You can use either of these to send automated emails or notify admins when bypasses occur. This allows you to maintain visibility without losing developer trust, since the bypass can be inspected and, if valid for cases like false positives, ignored. For cases where the bypass was not valid, admins can have conversations with the developer who bypassed the protection.

## Alerts are still created after bypassing push protection

Furthermore, even if a secret is bypassed during a push, GitHub will create a secret scanning alert, enabling admins to manage the bypassed secret appropriately. For example, automated token revocation can be enabled so that when secrets are detected in the repo post-push, automation can revoke the secret immediately for known token formats, or admins can be notified to check the bypass.

## Management by exception

This allows organization to "manage by exception" rather than "throttle by prevention". Ultimately this is a _cultural_ problem and not really a _technical_ problem. Organizations that demonstrate a "trust but verify" culture using the management techniques above will generally foster better developer experience and arguably end up being more secure than companies that promote a low-trust, hard gate.

Let's all remember to be good humans. Developers should sympathize with the IT and security teams - leaked credentials are a serious matter that could have large and far reaching negative consequences to companies. Developers need to be careful and thoughtful about preventing leaks. IT and security teams should in turn sympathize with developers, who are constantly under pressure to deliver more, faster - so anything that adds friction is going to be counterproductive. They should be careful and thoughtful of how they can partner with, rather than fight against, developers.

# Conclusion

Using GitHub Advanced Security secret scanning push protection is the best way for teams to effectively reduce the risk of credential leaks. While users can bypass push protections, there are valid reasons for this, and bypasses can be managed to ensure they are valid, while invalid bypasses can be mitigated quickly.

Happy push protecting!
