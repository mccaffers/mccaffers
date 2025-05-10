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

Driven by the fasination of the financial markets, I've been developing systems to analyise and trade the live markets. 

I started by building two backtesting engines in C# and C++. I'm now analysing and optimiing multiple trading strategies across diverse financial instruments (FOREX, Indices and Bonds), with a focus on building a robust live trading platform.

Additionally, I enjoy building and experimenting in different technologies and languages. I've developed several applications for Apple platforms (iOS and macOS). As an example, I built SearchOps, a iOS and macOS client for Elasticsearch (and Opensearch), which is open-source and on the Apple App Store. I've also experimented with geospatial data from OpenStreetMaps, extracting cities and ingesting into Elastcisearch, using Cloudflare Workers and KV to build <a href='https://geo.dev' target="_none">geo.dev<svg width="13.5" height="13.5" aria-hidden="true" viewBox="0 0 24 24" class="iconExternalLink_nPIU"><path fill="currentColor" d="M21 13v10h-21v-19h12v2h-10v15h17v-8h2zm3-12h-10.988l4.035 4-6.977 7.07 2.828 2.828 6.977-7.07 4.125 4.172v-11z"></path></svg></a>.

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