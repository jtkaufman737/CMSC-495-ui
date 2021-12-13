<template>
  <div class="about">
    <h5>UMGC Capstone Project</h5>
    <p>This UMGC capstone project was built in 2021, utilizing VueJS, Cytoscape.js, 
      Python, Flask, PostgreSQL, and psycopg2. It features a standalone API deployed to 
      heroku, and single page Vue app deployed to Netlify</p>
    <p>The application goal was to build fault tree analysis software, a system for 
      thinking about systems failures. This tool is used more often in disciplines outside of 
      software engineering but I ran across it while wishing I had a graphical tool to 
      debug a difficult problem.
    </p> 
    <p> "Symbols" in this app refer to different object types you can have in a chart, that represent
      types of events, or logical conditions such as AND and OR. These are helpful to map out cases like 
      if a is true and b is true, we see this bug. </p>
    <p>Future improvements include adding tests, making the UI more intuitive and fixing CSS, and I would have 
      liked to have button overlays on the chart elements so that adding a child or deleting the element 
      was done outside of the massive modal that is currently in place. Unfortunately, cytoscape.js renders this 
      graph as a canvas, so everything is dynamically drawn - it isn't a hierarchy of HTML nodes where you can 
      easily add a button. I <em>believe</em> I could add the appearance of buttons by having three canvases overlaid 
      and that will be an experiment I do when I get time.</p>
  </div>
</template>

<script>

export default {
  name: 'About',
}
</script>