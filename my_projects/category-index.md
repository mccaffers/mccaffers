---
sidebar_position: 999
title: "My Projects"
---


<style jsx>{`


  #box {
    display: flex;
    align-items: center;
  }
  
  #a {
    margin-right: 15px;
  }
  
  #b {
    flex-grow: 1;
  }
  
  .fourImages {
    width: 24%;
    margin-right: 1%;
  }
  
  .video-auto {
    border-radius: 8px;
  }
`}</style>

<div id="box">
  <div id="a">
    <img 
      src="/static/search_ops/logo.png" 
      style={{
        width: "55px",
        height: "55px",
        margin: 0,
        background: "none"
      }}
      alt="Search Ops Logo"
    />
  </div>
  <div id="b">
   <div style={{ fontSize: "24px", lineHeight:"20px"}}>Search Ops</div>
    <span style={{ fontSize: "14px" }}>iPhone, iPad and macOS</span>
  </div>
  <div id="buy">
    <a 
      href="https://apps.apple.com/us/app/search-ops/id6453696339?itsct=apps_box_link&itscg=30200" 
      target="_blank" 
      rel="noopener noreferrer"
      style={{ boxShadow: "none" }}
    >
      <div id="appleLogo"></div>
    </a>
  </div>
</div>

<div style={{ 
  border: "0px solid green", 
  padding: "10px 10px 0 10px", 
  marginBottom: "10px" 
}}>
  <p style={{ 
    fontWeight: 700, 
    marginTop: "5px", 
    marginBottom: 0 
  }}>
    Developer tool to query ElasticSearch and OpenSearch databases
  </p>
  <ul style={{ marginTop: "10px", marginBottom: "20px" }}>
    <li>Free text strings, using compounds (AND/OR) and Date Ranges</li>
    <li>View results as documents or in a table</li>
    <li>Easily switch between hosts, indexes and filter on mapped data types</li>
    <li>
      Read more about <a href="/search_ops/">Search Ops</a> & <a href="/search_ops/#release-notes">View Release Notes</a>
    </li>
  </ul>
</div>

<div style={{ textAlign: "left" }}>
  <img 
    className="fourImages" 
    src="/static/search_ops/listview.png" 
    loading="lazy"
    alt="List View" 
  />
  <img 
    className="fourImages" 
    src="/static/search_ops/tableview.png" 
    loading="lazy"
    alt="Table View" 
  />
  <img 
    className="fourImages" 
    src="/static/search_ops/queryfilter.png" 
    loading="lazy"
    alt="Query Filter" 
  />
  <img 
    className="fourImages" 
    src="/static/search_ops/document.png" 
    loading="lazy"
    alt="Document View" 
  />
</div>

<div id="box" style={{ marginTop: "30px" }}>
  <div id="a">
    <img 
      src="/static/alfie-the-greedy-fish/alfie-the-greedy-fish.png" 
      style={{
        width: "55px",
        height: "55px",
        margin: 0,
        background: "none"
      }}
      alt="Alfie The Greedy Fish Logo"
    />
  </div>
  <div id="b">
   <div style={{ fontSize: "24px", lineHeight:"20px"}}>Alfie The Greedy Fish</div>
    <span style={{ fontSize: "14px" }}>iPhone & iPad</span>
  </div>
  <div id="buy">
    <a 
      href="https://apps.apple.com/us/app/alfie-the-greedy-fish/id541365811?itsct=apps_box_link&itscg=30200" 
      target="_blank" 
      rel="noopener noreferrer"
      style={{ boxShadow: "none" }}
    >
      <div id="appleLogo"></div>
    </a>
  </div>
</div>

<div style={{ 
  border: "0px solid green", 
  padding: "10px 10px 0 10px" 
}}>
  Alfie The Greedy Fish is an Arcade Game. You play Alfie, a small red fish, navigating the sea to eat all fish, and you must avoid the sharks! There are four game modes, Arcade, Survival, Deep Sea and Frenzy.
</div>

<div style={{ textAlign: "left", marginTop: "10px" }}>
  <video 
    loop={true} 
    autoPlay={true} 
    muted={true} 
    playsInline={true} 
    className="video-auto rounded" 
    width="100%"
  >
    <source src="/static/alfie-the-greedy-fish/app-store-video.mp4" type="video/mp4" />
    Your browser does not support video playback.
  </video>
</div>

<div id="box" style={{ marginTop: "30px" }}>
  <div id="a">
    <img 
      src="/static/space-academy/logo.png" 
      style={{
        width: "55px",
        height: "55px",
        margin: 0,
        background: "none"
      }}
      alt="Moon Defence Logo"
    />
  </div>
  <div id="b">
   <div style={{ fontSize: "24px", lineHeight:"20px"}}>Moon Defence</div>
    <span style={{ fontSize: "14px" }}>iPhone & iPad</span>
  </div>
  <div id="buy">
    <a 
      href="https://apps.apple.com/us/app/space-academy/id604047933?itsct=apps_box_link&itscg=30200" 
      target="_blank" 
      rel="noopener noreferrer"
      style={{ boxShadow: "none" }}
    >
      <div id="appleLogo"></div>
    </a>
  </div>
</div>

<div style={{ 
  border: "0px solid green", 
  padding: "10px 10px 0 10px" 
}}>
  Moon Defence is a space arcade game. You must defend the spaceships on the moon from incoming asteroids.
</div>

<div style={{ textAlign: "left", marginTop: "10px" }}>
  <video 
    loop={true} 
    autoPlay={true} 
    muted={true} 
    playsInline={true} 
    className="video-auto rounded" 
    width="100%"
  >
    <source src="/static/space-academy/app-store.mp4" type="video/mp4" />
    Your browser does not support video playback.
  </video>
</div>
