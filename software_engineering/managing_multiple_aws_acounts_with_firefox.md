---
sidebar_position: 1
title: "Managing multiple AWS accounts with Firefox Containers"
description: "abc"
---

<img src="/static/firefox-containers/landing.png"  />

## Multiple Accounts 

First of all, I want to explain the benefits of using multiple AWS accounts. By using an AWS account per project you get:

* Separation of concern and group up workloads together

    * You know everything in the AWS Account relates to one project, keeping the use of services clean and easy to navigate. I've often found Lambdas, S3, and CloudWatch to become very noisy as you start to combine projects, making it difficult to navigate and troubleshoot.

* Breakdown of billing per project

    * Get a monthly bill broken down by project, and via the Organisational Unit, you can see a high level of all costs across all subscriptions.

* Limit the scope of impact from adverse events

    * Know as you use IAM and Security Credentials that they are only able to access one project

AWS also recommends it: 

> Using multiple AWS accounts to help isolate and manage your business applications and data can help you optimize across most of the AWS Well-Architected Framework pillars, including operational excellence, security, reliability, and cost optimization. https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html

### Adding a new account

It's fairly simple to add new AWS accounts via the AWS Organizations Service. From there you can manage the users who can access the AWS Account via the IAM Identity Management Center.

<img src="/static/firefox-containers/addawsaccount.png"  />


## Switching between acounts

I've found it difficult manage multiple accounts though, often logging in and out of accounts in the same browser to use different services. Authentication cookies are shared between tabs so it becomes a bit of a nightmare.

However I've discovered that the FireFox Team at Mozzila have made an extension for the Firefox Browser so that cookies are contained within the tab, allowing you to have different sessions in different tabs.

> Firefox Multi-Account Containers lets you keep parts of your online life separated into color-coded tabs. Cookies are separated by container, allowing you to use the web with multiple accounts https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers/


Example of having multiple AWS Accounts open in the same browser window:

   <div style={{
      textAlign: 'left',
      marginBottom: '20px',
      display: 'flex',
      flexWrap: 'wrap',
      gap: '20px'
    }}>
      <div style={{
        flex: '1 1 300px',
        minWidth: '250px'
      }}>
        <img 
          src="/static/firefox-containers/tab2.png" 
          alt="AWS Account for SearchOps"
          style={{ width: '100%', height: 'auto' }}
        />
        <div style={{
          textAlign: 'center',
          fontSize: '14px',
          marginTop: '8px'
        }}>AWS Account for SearchOps</div>
      </div>
      <div style={{
        flex: '1 1 300px',
        minWidth: '250px'
      }}>
        <img 
          src="/static/firefox-containers/tab1.png" 
          alt="AWS Account for Trading Experiments"
          style={{ width: '100%', height: 'auto' }}
        />
        <div style={{
          textAlign: 'center',
          fontSize: '14px',
          marginTop: '8px'
        }}>AWS Account for Trading Experiments</div>
      </div>
    </div>

There are a few customisation options with the the icons and coloring the tabs, but it becomes really powerful when you add a further extension 'open-url-in-container'.

> This is a Firefox extension that enables support for opening links in specific containers using custom protocol handler. It works for terminal, OS shortcuts, bookmarks, password managers, regular HTML pages and many other things. https://github.com/honsiorovskyi/open-url-in-container

## Open multiple containers on launch

With this extension, you can set Firefox to open with multiple tab containers with multiple AWS accounts. This smoothens the experience of using multiple accounts and instantly being able to jump into all of them.

As an example, this opens up a tab in the `SearchOps`` container with it’s own AWS account (which I’m using to build ElasticSearch test infrastructure).

```
ext+container:name=SearchOps&url=https://us-east-1.console.aws.amazon.com
```

In Firefox settings, you can combine multiple tabs together with a pipe `|`:

```
ext+container:name=SearchOps&url=https://us-east-1.console.aws.amazon.com|ext+container:name=AWSMain&url=https://us-east-1.console.aws.amazon.com
```

<img src="/static/firefox-containers/settings.png" />

## Bookmarks

You can also put these urls into your bookmarks, so you open up independent containers for each AWS session.

<img src="/static/firefox-containers/bookmarks.png" />

## Video demo

Launching Firefox with two AWS accounts in two container tabs

<img src="/static/firefox-containers/output.webp"  />

## Quick Download links

Firefox https://www.mozilla.org/en-GB/firefox/new/

Multi Account Extension https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers/

Open links in container Extension https://addons.mozilla.org/en-GB/firefox/addon/open-url-in-container/

## Conclusion

Using Firefox with multiple containers to management AWS Accounts is super useful and can optmise your workflow. 

Also, managing AWS Accounts is a good example, but this workflow would support other usecases to login to multiple sites in the same browser.