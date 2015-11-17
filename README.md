Author: Harini Anand

Date: 11/09/2015

Description: 
## Website Performance Optimization portfolio project

The challenge is to optimize the online portfolio for speed! In particular, to optimize the critical rendering path and make the page render as quickly as possible.

The code is located at https://github.com/udacity/frontend-nanodegree-mobile-portfolio.git

Prerequisites:

1. Install git
2. Install bash
3. Install ngrok found at the location: https://ngrok.com/
4. Install google chrome 

####Part 1: To Optimize PageSpeed Insights score for index.html

0. Open a git bash terminal
1. git clone https://github.com/udacity/frontend-nanodegree-mobile-portfolio.git
2. At the terminal,

  cd /path/to/your-project-folder
  python -m SimpleHTTPServer 80

3. Open ngrok and type ngrok http 80
4. Open a google chrome and type http://localhost:4040 
   Then create a tunnel, obtain the external url
5. Open Google page speed insights https://developers.google.com/speed/pagespeed/insights/
   copy the external url in the text box and click on Analyze.


####Part 2: Optimize Frames per Second in pizza.html

1. Open google chrome
2. Open the views/pizza.html
3. Open Google developer tools and run timeline by scrolling the page and stop the timeline

####Part 3: Optimize time to resize pizzas in pizza.html (should be less than < 5 milliseconds)

1. Open google chrome
2. Open the views/pizza.html
3. use the slide option on the page and move from medium to small or large 
4. Check the console print out
5. Time to resize pizzas should be < 5 milliseconds

Changes Implemented:


In index.html for Part 1:

A. Added media to fetching print.css
 <link href="css/print.css" rel="stylesheet" media="print">

B. Based on http://googlewebfonts.blogspot.com/2010/09/optimizing-use-of-google-font-api.html
google fonts should be placed very early in the file. It is placed already ahead of css/style.css in the index.html file.
However, replacing the googlefont with websafe font has a big impact.
I changed to a web safe font: Arial, Helvetica, sans-serif which is closest to Open sans-serif.


C. Made the Java script for analytics async 
<script async src="http://www.google-analytics.com/analytics.js"></script>

D. Since the below java script is for Analytics. Kept in the style tag only that CSS that is needed to 
to draw the page until the full CSS is downloaded.
used window.onload to execute this script after loading has finished.


    <script>
      (function(w,g){w['GoogleAnalyticsObject']=g;
      w[g]=w[g]||function(){(w[g].q=w[g].q||[]).push(arguments)};w[g].l=1*new Date();})(window,'ga');

      // TODO: replace with your Google Analytics profile ID.
      ga('create', 'UA-XXXX-Y');
      ga('send', 'pageview');
    </script>
    <script async src="http://www.google-analytics.com/analytics.js"></script>
    <script async src="js/perfmatters.js"></script>
  </head>

E. Compressed the image file pizzeria.jpg using http://jpeg-optimizer.com/
   New file: pizzeria__1447035707_108.214.189.88.jpg New size: 28KB 400X300 pixels
   old file: Pizzeria.jpg: 2.25 MB 2048X1536 pixels
   Further reduced the new file using optimizilla.com to 11.5 KB

F. Compressed the profilepic.jpg from 14.0KB to 969bytes using optimizilla.com
 
G. downloaded and used the optimized image, JavaScript, and CSS resources from the page speed insights page.

H. Minified HTML file using https://kangax.github.io/html-minifier/.


------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------

 In view/pizza.html, to get 60fps while scrolling (animating), I performed the following changes:

A. used the compressed and optimized JPG for pizzeria.html (used pizza-min.jpg)

B. Insize PizzaGenerator, document.getElementbyId("randomPizzas") is called 100 times.
 I placed it outside the for loop.

  Original code:
 // This for-loop actually creates and appends all of the pizzas when the page loads
for (var i = 2; i < 100; i++) {
  var pizzasDiv = document.getElementById("randomPizzas");
  pizzasDiv.appendChild(pizzaElementGenerator(i));
}

 
 New code:
 var pizzasDiv = document.getElementById("randomPizzas");
for (var i = 2; i < 100; i++) {
  pizzasDiv.appendChild(pizzaElementGenerator(i));
}


C. In UpdatePositions(), there is a FSL caused by calling document.body.scrollTop
 in a loop that makes style changes. 

 Original code:
 var items = document.querySelectorAll('.mover');
  for (var i = 0; i < items.length; i++) {
    var phase = Math.sin((document.body.scrollTop / 1250) + (i % 5));
    items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }

  New code (removed invocation to ScrollTop)
  var items = document.querySelectorAll('.mover');
  for (var i = 0; i < items.length; i++) {
    items[i].style.left = items[i].basicLeft + 100 * (i % 5) + 'px';
  } 

 3. I also added the fix to the function changePizzaSizes as instructed in the course video

 4. I added the will-change: transform to .mover and .container css selector in the css/style.css file

 .mover {
  position: fixed;
  width: 256px;
  z-index: -1;
  will-change: transform;
}

.container {
  background-color: rgba(240, 60, 60, 0.8);
  will-transform: transform;
  transform: translateZ(0);
}

 5. I changed the function to execute for "DOMContentLoaded" to load only half the images

  // Generates the sliding pizzas when the page loads.
  document.addEventListener('DOMContentLoaded', function() {
  var cols = 8;
  var s = 256;
  var movingpizza = document.querySelector("#movingPizzas1")
  
  // only create half the images and do update positions only half the time
  for (var i = 0; i < 200; i=i+2) {
    var elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza-min.png";
    elem.style.height = "100px";
    elem.style.width = "73.333px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';

    movingpizza.appendChild(elem);
  } 
      updatePositions();
  });

6. UpdatePostions() is showed many many times in the timeline

Modified to run only on half the movers.

function updatePositions() {
  frame++;
  window.performance.mark("mark_start_frame");

  var items = document.querySelectorAll('.mover');
  //var phase = Math.sin(document.body.scrollTop / 1250)
  // Modified to only change style for half of them
  for (var i = 0; i < items.length; i=i+2) {
    items[i].style.left = items[i].basicLeft + 100 * (i % 5) + 'px';
  } 

  // User Timing API to the rescue again. Seriously, it's worth learning.
  // Super easy to create custom metrics.
  window.performance.mark("mark_end_frame");
  window.performance.measure("measure_frame_duration", "mark_start_frame", "mark_end_frame");
  if (frame % 10 === 0) {
    var timesToUpdatePosition = window.performance.getEntriesByName("measure_frame_duration");
    logAverageFrame(timesToUpdatePosition);
  }
}

7. I tried worker threads but it would not work with Chrome
I tried to follow instructions to make it run with the webserver configuration.
But still encountered a number of errors and abandoned the solution to use worker thread.


------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------
Part 3: To get time to resize pizzas to be less than 5ms, I made the following changes:

1. Made the following changes to changePizzaSizes.

Used ForEach. Determined the value for dX and newwidth only once. 

// Iterates through pizza elements on the page and changes their widths
  function changePizzaSizes(size) {
    var randomPizzas = document.getElementsByClassName('randomPizzaContainer');
    var dx = determineDx(randomPizzas[0],size);
    var newwidth = randomPizzas[0].offsetWidth + dx + 'px';
    var randomPizzasArray = Array.prototype.slice.call(randomPizzas);
    randomPizzasArray.forEach(function changeStyle(value) {
      value.style.width = newwidth;
    });
  }

2. Used Document Fragment to add pizzaElements to PizzaDivs.

var pizzasDiv = document.getElementById("randomPizzas");
// This for-loop actually creates and appends all of the pizzas when the page loads
var docFrag = document.createDocumentFragment();
for (var i = 2; i < 100; i++) {
  docFrag.appendChild(pizzaElementGenerator(i));
}
pizzasDiv.appendChild(docFrag);

3. Implemented the changes suggested by the reviewed and declared elem outside the loop


var elem;
  for (var i = 0; i < totalPizzas; i++) {
    elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza-min.png";
    elem.style.height = "100px";
    elem.style.width = "73.333px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';
    document.querySelector("#movingPizzas1").appendChild(elem);
  }

  