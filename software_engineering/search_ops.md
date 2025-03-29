---
sidebar_position: 2
title: "Search Ops - iOS app to search ElasticSearch"
description: "Query ElasticSearch from your iPhone"
keywords: ["Query ElasticSearch", "OpenSearch App", "ElasticSearch App", "Mobile Kibana"]
tags: ["Software Engineering"]
authors: 
  - name: John Doe
    title: Software Engineer
    url: https://github.com/johndoe
    image_url: https://github.com/johndoe.png
---

Over the last few years, I've been building trading strategies and outputting results to ElasticSearch. I wanted to view my results on the go and have observability over my infrastructure. While Kibana (and OpenDashboard) are excellent tools, they cater to a full desktop experience with all the bells and whistles. 

So I made Search Ops, a mobile app that enables you to query and view your data from ElasticSearch.

<div style={{textAlign: "left"}}>
<img className="fourImages" src="/static/search_ops/listview.png" loading="lazy" alt="List view" />
<img className="fourImages" src="/static/search_ops/tableview.png" loading="lazy" alt="Table view" />
<img className="fourImages" src="/static/search_ops/queryfilter.png" loading="lazy" alt="Query filter" />
<img className="fourImages" src="/static/search_ops/document.png" loading="lazy" alt="Document" />
</div>

## Available on Apple's App Store

* Now available on the Apple App Store for iPhone and iPad with a one time fee
* Designed for iOS 16.0 and above
* Privacy-focused app offers powerful search operations with no analytics or tracking
* Open-source business logic available on GitHub 

<a href='https://apps.apple.com/us/app/search-ops/id6453696339?itsct=apps_box_link&itscg=30200' target="_blank" rel="noopener noreferrer" style={{boxShadow: "none", marginTop: "10px"}}>
        <div id="appleLogo"></div>
      </a>

## Features

The app allows you to query ElasticSearch and OpenSearch clusters using Free Text String, with compounds (AND/OR) and with Date Ranges (if you have a defined a Date type in your index). Queries are saved, so you can easily switch between hosts and indexes, reusing queries whereever you need to.

The application support ElasticSearch (v5.0 and above) and OpenSearch (v1.0 and above)

Connections can be made using a CloudID from Elastic.co or a direct host connection (HTTP/HTTPS), with a range of authentication methods.
* Username/Password
* Auth Token
* API Token
* API Key

The application provides a read only access to your Search clusters, and as a result only needs a user with Viewer and Monitor User.

<div style={{width: "100%", textAlign: "left"}}>
<img className="searchOpsImages" src="/static/search_ops/queryfilter.png" alt="Query filter" />
<img className="searchOpsImages" src="/static/search_ops/document.png" alt="Document" />
</div>

## Business Logic

The application is split into two parts, the Business Logic (a Swift Package) and a Presentation Layer using SwiftUI. The Business Logic is public and available on Github to view. The app uses a local on device open source database called Realm, by MongoDB foundation with encyrption on.

<img style={{width: "100%", backgroundColor: "rgb(255, 255, 255)"}}  src="/static/search_ops/businesslogic.svg" alt="Business logic" />

The Business Logic is available to view on [Github](https://github.com/mccaffers/search-ops)

## Encryption

Search Ops uses Realm as local on device database with encyrption turned on. You can see the full implementation here, [SearchOps/Sources/SearchOps/DataManagers/RealmManager.swift](https://github.com/mccaffers/SearchOps/blob/main/Sources/SearchOps/DataManagers/RealmManager.swift).

The `RealmManager Class` generates a key and saves it to the local keychain. This keychain does not persist to iCloud.

```swift
private static func getKey() throws -> Data {
    // generate key
    var key = Data(count: 64)
    try key.withUnsafeMutableBytes({ (pointer: UnsafeMutableRawBufferPointer) in
        let result = SecRandomCopyBytes(kSecRandomDefault, 64, pointer.baseAddress!)
    })
    return key
}
```

The Realm Database is opened with the encryption key thats unique to the local device.

```swift 
private static func getRealmConfig() -> Realm.Configuration { 
    // try to load configuration with key
    return !try Realm.Configuration(encryptionKey: getKey())
}
```

## History and Logs

You can reuse previous queries and view the response of any requests made.  

<div style={{width: "100%", textAlign: "left"}}>
<img className="searchOpsImages" src="/static/search_ops/requestlogs.png" alt="Request logs" />
<img className="searchOpsImages" src="/static/search_ops/requestlogdetail.png" alt="Request log detail" />
</div>

## Privacy

Privacy is important and I’ve taken extra steps to build trust, splitting the application up and sharing the business logic.

The Business Logic documents how host credentials are stored locally, and requests are made. The commit hash of the business logic in use is showned in the iOS application on the Settings page. There are also no analytics or tracking within the application too.

## Release Notes

I've made a website just for the SearchOps app, to showcase new features and publish release notes, see more at [https://searchops.app](https://searchops.app/release-notes/)
