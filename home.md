---
id: home
slug: /
sidebar_position: 0
sidebar_class_name: hidden
title: Home
---

<style>{`  
  h1 {
    display:none;
  }
  .breadcrumbs {
    display: none;
  } 
  .theme-doc-footer.docusaurus-mt-lg {
    display:none;
  }
  .pagination-nav.docusaurus-mt-lg {
    display:none;
  }
  @media screen and (max-width: 700px) {
    .image-container {
      flex-direction: column !important;
    }
    .image-container div {
      width: 100% !important;
    }
  }
`}</style>

<h2 style={{marginTop:0}} >Hi, I'm Ryan! 👋 </h2>

Driven by the fasination fo the markets, I'm developing trading systems. I started by building two backtesting engines in C# and C++. I'm now analysing and optimiing multiple trading strategies across diverse financial instruments (FOREX, Indices and Bonds), with a focus on building a robust live trading platform.

Additionally, I enjoy building native applications for Apple platforms (iOS and macOS), experimenting and prototyping. I've built SearchOps, powering mobile querying of Elasticsearch, and a web development prototype of geospatial data, hosted on geo.dev.

<h2 style={{marginTop:0}} >My Projects</h2>

<div className="image-container" style={{ display: 'flex', justifyContent: 'space-between', gap: '10px', alignItems: 'center' }}>
  <div 
    style={{ 
      width: '49%', 
    }} 
  >
  <div>Quantitative Analysis</div>
    <div 
    style={{ 
      width: '100%', 
      aspectRatio: '16/9',
      backgroundImage: 'url(/static/randomly_trading/dark/random-indices-sp500-variable.svg)',
      backgroundSize: 'cover',
      backgroundPosition: 'center',
      borderRadius: '5px'
    }} 
  ></div>
  </div>
  <div 
    style={{ 
      width: '49%', 
    }} 
  >
  <div>SearchOps for iOS and macOS</div>
    <div 
    style={{ 
      width: '100%', 
      aspectRatio: '16/9',
      backgroundImage: 'url(/static/home_images/macos_main_search.png)',
      backgroundSize: 'cover',
      backgroundPosition: 'center',
      borderRadius: '5px'
    }} 
  ></div>
  </div>
</div>

<div className="image-container" style={{ display: 'flex', justifyContent: 'space-between', gap: '10px', alignItems: 'center', marginTop:"20px" }}>

  <div 
    style={{ 
      width: '49%', 
    }} 
  >
  <div>Alfie the greedy fish (iOS)</div>
 <video
    loop
    autoPlay
    playsInline
    className="video-auto rounded"
    width="100%"
  >
    <source src="/static/alfie-the-greedy-fish/app-store-video.mp4" type="video/mp4" />
    <p>Your browser does not support video playback</p>
  </video>
  </div>

  <div 
    style={{ 
      width: '49%', 
    }} 
  >
  <div>Moon Defence (iOS)</div>
 <video
    loop
    autoPlay
    playsInline
    className="video-auto rounded"
    width="100%"
  >
    <source src="/static/space-academy/app-store.mp4" type="video/mp4" />
    <p>Your browser does not support video playback</p>
  </video>
  </div>
</div>