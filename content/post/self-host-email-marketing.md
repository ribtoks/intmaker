---
title: How to self-host email marketing list if you are a developer
date: 2020-02-20T18:22:19-08:00
author: "Taras Kushnir"
image: emails.png
categories:
  - Programming
keywords:
  - email
  - marketing
  - aws
  - lambda
---

Do you know how expensitve is to have email marketing list? Most of the providers have a 1000 subscribers free tier, but the more subscribers you have, the more you have to pay. I'm running few side projects and always wanted them to have their own subscribers list, but I'm not ready to pay $30-70 per month to MailChimp or others. Instead I would more prefer something like $0 per month and unlimited number of subscribers forever. If you want to know how I achieved that, read on.

<!-- more -->

## The search

### Paid providers

So initially I decided I don't want to spend any time for managing server infrastructure or anything alike. I will just buy cheapest email marketing provider possible and will stay with it. I was slightly disappointed, that even "cheapest" ones were not cheap enough for me.

**Provider** | **Free tier** | **Price for 10k** | **Alexa ranking**
----- | ----- | ----- | -----
[EmailOctopus](https://emailoctopus.com/pricing) | 2500 contacts | $20-30/month | 27,452
[MailChimp](https://mailchimp.com/pricing/) | 2500 contacts | $74+/month | 440
[MoonMail](https://moonmail.io/pricing/) | Self-hosted | $149+/month | 171,413
[MailerLite](https://www.mailerlite.com/pricing) | 1000 contacts | $50/month | 4,360
[Sendy](https://sendy.co/) | Self-hosted | $56 1-time fee | 27,195
[MadMimi](https://madmimi.com/service_agreements/choose_plan) | - | $42/month | 10,446
[EasySendy](https://easysendy.com/pricing/email-campaigns/) | - | $19/month | 75,410

This looked slightly disappointing, given that cheaper providers had quite limited funtionality comparetively to more expensive ones. So my next logical step was to check how are free, OSS and self-hosted solutions doing.

### Free, OSS solutions

Quick googling revealed the following list: [tinycampaign](https://github.com/parkerj/tinycampaign), [audience](https://github.com/aniftyco/audience), [mail-for-good](https://github.com/freeCodeCamp/mail-for-good), [colossus](https://github.com/vitorfs/colossus).

All these contenders have few cons for me:

* you need to run a web-server all of the time, which means you have to maintain it, install updates, provision certificates and so on. Also it is not free
* these solutions are all-in-one solutions, providing email templates, analytics, A/B testing and email sending

It was quite demotivating. These solutions tried to implement everything instead of just email list. And I couldn't understand why do you need to run a web-server all of the time, given that in 2020 you have all the magic of serverless. But none of these systems took advantage of it. None, but one.

So I found [MoonMail](https://moonmail.io/) which looked like a system you would want to use. It was running on AWS stack on serverless architecture. No wasting resources, it supported email templates, analytics and A/B testing out of the box. **It looked really clean and nice with only one problem: no documentation.** [Install docs](https://github.com/MoonMail/MoonMail#getting-started) looked simply insane. There're 10 services you have to deploy separately, half of which require exotic legacy serverless framework version and half uses new one. There's no documentation about which configs you should edit and which not. This is understandable because they want to sell it like EmailOctopus did, transitioning from open source software to SaaS. It's even not trivial to find GitHub link on their website.

So I really had no choice but to create this system on my own.

## The project

I spent a week and created first version of email marketing system that I called *Listing*. It manages arbitrary amount of subscription lists using few lambda functions written in Go. Subscriptions are stored in a DynamoDB table and are managed by a lambda function with endpoints `/subscribe`, `/unsubscribe`, `/confirm` and `/subscribers`. The last endpoint is protected with BasicAuth for "admin" access (export/import of subscribers). Bounces and Complaints are stored in additional DynamoDB table. This table is managed by another lambda function that is subscribed to SNS topic. AWS SES for a specific domain is a publisher to this SNS topic.

*Listing* uses so-called "double-confirmation" system where user has to confirm it's email even after entering it on your website and pressing "Subscribe" button. The reason is that this is way more reliable in terms of loyal audience building.

Usage on the website is absolutely trivial. After lambda function is deployed, you get a nice `/subscribe` API that you can add to a form like this:

```
<form action="https://abcdef123456.execute-api.us-east-1.amazonaws.com/prod/subscribe" method="post">
  <div>
      <input type="text" id="name" name="name" placeholder="Your name">
  </div>
  <div>
      <input type="email" id="email" name="email" placeholder="Your email" required>
  </div>
  <div>
      <input class="button" type="submit" value="Subscribe">
  </div>
  <input type="hidden" id="newsletter" name="newsletter" value="NewsletterName">
</form>
```

## Operation

Have I mentioned *Listing* can be deployed with only one command `serverless deploy`? After this it just works. But that's not all. *Listing* serves couple of ordinary http endpoints and while you can do all operations using `curl`, I created `listing-cli` command line application that simplifies it.

For example, to subscribe to a newsletter, you can run:

```
listing-cli -mode subscribe -email foo@bar.com -newsletter NewsletterName -url https://qwerty12345.execute-api.us-east-1.amazonaws.com/dev
```

or to list all subscribers:

```
listing-cli -auth-token your-token-here -url "https://qwerty12345.execute-api.us-east-1.amazonaws.com/dev" -mode export -newsletter Listing1
```

One of the functions of `listing-cli` is import and export of the subscription list. You may want to do it for backups, when you move your database or to send actual emails.

Even though in reality initial deployment is a little bit more compliated than `serverless deploy` (you have to create nice subscribed/unsubscribed pages and so on), this system is totally hassle-free and autonomous. Beside that it costs you nothing. AWS Lamda provides first 1 million invocations free each month and subscription lambda of *Listing* is small and fast. That means that most probably your marketing list will never grow so fast that you will ever have to pay for it.

## Sending emails

There's one important piece left in this puzzle: *sending the actual emails*. If you never done that, here's what you need:

* email templates that will allow to personalize contents
* compute: smpt server that will send all those emails
* analytics: you may want to know how successful your campaign was
* software that will evaluate/execute those templates with your actual users and send the emails through your server

### Templates

Getting templates was actually the easiest part. I spent around an hour with [mosaico](http://mosaico.io/) and [beefree](https://beefree.io/) but didn't manage to compile a template that I would like. But I'm not a designer either. So I ended up buying a pack on Themeforest for $20. Even though it's not free, it will stay with me for a while and it contains around 70 email variations from which I will have around 10 useful ones.

### Compute

Since we are leveraging AWS stack, it might be interesting to know why people actually use AWS SES (Simple Email Service) for emails. And the reaons is the price: you pay 10 cents for 1000 emails, but if you send emails from EC2 instance, first 60k emails per month are free. That ultimately means **you can send 60k emails for free and then 10k emails for $1**. That is not exactly free, but it is very close to it.

So the goal is to somehow use AWS SES for our email sending. Systems like Moonmail, Sendy and EmailOctopus allow you to use SES as their backend for actual sending. But you still have to pay for the system itself.

**Good news is that SES provides you with SMTP endpoint** so if you just set it up on your local machine as outgoing server, you can send all emails from your favorite mail app. Of course, it's not what you would do if you have a 1000 of subscribers, but it allows to have a very neat automation around.

### Analytics

Although I'm not a fan of Google, but it's hard to beat their free Google Analytics tier. It allows 30 million events per property per month which is definitely more than enough for email campaigns.

Usage of Google Analytics is also trivial. Just add a "tracking pixel" that implements [Measurement Protocol](https://developers.google.com/analytics/devguides/collection/protocol/v1/devguide) to the bottom of your email and you are done.

```
<img src="https://www.google-analytics.com/collect?v=1&tid=UA-12345678-90&t=event&ec=email&ea=open&cm=email&ci=campaignID"/>
```

In order to track link clicks you can add something like `utm_source=email&utm_campaign=YourCampaign` and hope that your subscribers don't use ClearURLs add-on in browser.

### Software

This is the interesting part. **So the goal was just to find a piece of software that would allow creating html templates and sending them to SMTP server.** Initially I found [paperboy](https://github.com/rykov/paperboy) and [thunder mail](https://github.com/Circle-gg/thunder-mail) to be good candidates for the software. However, there are some limitations with them too. Paperboy has been sort-of-abandoned for some time and some useful features like emails preview were pending in branches. Thundermail required to write Javascript scripts in order to send the emails and I only wanted to create templates.

Since the only functionality you actually need is to execute templates and send emails, I implemented this as well. I was inspired by paperboy system, but made it a little bit easier. Paperboy creates a project with hierarchy like [hugo](https://gohugo.io/) does: `./contents/`, `./lists/`, `./layouts` and others. In *Listing* I'm just passing template and subscribers list as parameters allowing user to organize the "project" on their own. Templates are ordinary Go templates so **all functionality was already there and I only had to put it together**. This is how `listing-send` helper was created.

## Workflow

Typical workflow for *Listing* was to download all subscribers with `listing-cli` and then, after building a template offline, send the emails using `listing-send`. Of course you have to send test emails first and also to check if it works on mobile.

I keep all campaigns and subscriber lists in Git so every campaign is actually reproducible.

## Afterword

It was a great fun building [listing](https://github.com/ribtoks/listing) and actually using it. Now my side-project Xpiks [has it's own marketing list](https://xpiksapp.com/blog/2020/newsletter/) and I already sent my first newsletter to the first 100 subscribers! Except of the templates, I didn't pay a penny to this point and even though it looked like a bit of "not-invented-here" syndrome, it worked out well and paid off.

Links:

* [listing source code](https://github.com/ribtoks/listing)
* [Endpoints docs](https://github.com/ribtoks/listing/blob/master/docs/ENDPOINTS.md)
* [Deployment docs](https://github.com/ribtoks/listing/blob/master/docs/DEPLOYMENT.md)
