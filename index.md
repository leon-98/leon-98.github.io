<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1"><!-- Begin Jekyll SEO tag v2.7.1 -->
<title>FRP Asteroids | Tim’s code stuff</title>
<meta name="generator" content="Jekyll v3.9.0" />
<meta property="og:title" content="FRP Asteroids" />
<meta property="og:locale" content="en_US" />
<meta name="description" content="Introduction Functional Reactive Programming (specifically the Observable/Observer pattern) allows us to capture asynchronous actions like user interface events in streams. These allow us to “linearise” the flow of control, avoid deeply nested loops, and process the stream with pure, referentially transparent functions." />
<meta property="og:description" content="Introduction Functional Reactive Programming (specifically the Observable/Observer pattern) allows us to capture asynchronous actions like user interface events in streams. These allow us to “linearise” the flow of control, avoid deeply nested loops, and process the stream with pure, referentially transparent functions." />
<link rel="canonical" href="https://tgdwyer.github.io/asteroids/" />
<meta property="og:url" content="https://tgdwyer.github.io/asteroids/" />
<meta property="og:site_name" content="Tim’s code stuff" />
<meta name="twitter:card" content="summary" />
<meta property="twitter:title" content="FRP Asteroids" />

<!-- End Jekyll SEO tag -->
<link rel="shortcut icon" href="https://tgdwyer.github.io/favicon.ico" />
    <link rel="stylesheet" href="/assets/main.css">
    <link rel="stylesheet" href="/assets/css/styles.css"><link type="application/atom+xml" rel="alternate" href="https://tgdwyer.github.io/feed.xml" title="Tim's code stuff" /><!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-159840333-1"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-159840333-1');
</script></head><body><header class="site-header" role="banner">

  <div class="wrapper"><a class="site-title" rel="author" href="/">Tim&#39;s code stuff</a><nav class="site-nav">
        <input type="checkbox" id="nav-trigger" class="nav-trigger" />
        <label for="nav-trigger">
          <span class="menu-icon">
            <svg viewBox="0 0 18 15" width="18px" height="15px">
              <path d="M18,1.484c0,0.82-0.665,1.484-1.484,1.484H1.484C0.665,2.969,0,2.304,0,1.484l0,0C0,0.665,0.665,0,1.484,0 h15.032C17.335,0,18,0.665,18,1.484L18,1.484z M18,7.516C18,8.335,17.335,9,16.516,9H1.484C0.665,9,0,8.335,0,7.516l0,0 c0-0.82,0.665-1.484,1.484-1.484h15.032C17.335,6.031,18,6.696,18,7.516L18,7.516z M18,13.516C18,14.335,17.335,15,16.516,15H1.484 C0.665,15,0,14.335,0,13.516l0,0c0-0.82,0.665-1.483,1.484-1.483h15.032C17.335,12.031,18,12.695,18,13.516L18,13.516z"/>
            </svg>
          </span>
        </label>


</header>
<main class="page-content" aria-label="Content">
      <div class="wrapper">
        <article class="post">

  <header class="post-header">
    <h1 class="post-title">FRP Asteroids</h1>
  </header>

  <div class="post-content">
    <h2 id="introduction">Introduction</h2>
<p>Functional Reactive Programming (specifically the Observable/Observer pattern) allows us to capture asynchronous actions like user interface events in streams.  These allow us to “linearise” the flow of control, avoid deeply nested loops, and process the stream with pure, referentially transparent functions.</p>

<p>As an example we will build a little “Asteroids” game using FRP.  We’re going to use <a href="https://rxjs-dev.firebaseapp.com/">rxjs</a> as our Observable implementation, and we are going to render it in HTML using SVG.
We’re also going to take some pains to make pure functional code (and lots of beautiful curried lambda (arrow) functions). We’ll use <a href="https://www.typescriptlang.org/">typescript type annotations</a> to help us ensure that our data is indeed immutable and to guide us in plugging everything together without type errors into a nicely decoupled <a href="https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller">Model-View-Controller (MVC) architecture</a>:</p>

<p><img src="GeneralMVC.png" alt="MVC Architecture" /></p>

<p>If you’re the kind of person who likes to work backwards, <a href="https://asteroids05.stackblitz.io/">you can jump straight to playing the final result</a> and you can also <a href="https://stackblitz.com/edit/asteroids05?file=index.ts">live edit its code</a>.</p>

<p>We’ll build it up in several steps.</p>
<ul>
  <li>First, we’ll just <a href="#rotating-the-ship">rotate the ship</a>
    <ol>
      <li><a href="#using-events-directly">directly with old-school events</a></li>
      <li><a href="#using-observable">abstracting away event handling with an Observable</a></li>
    </ol>
  </li>
  <li>Then, we’ll <a href="#pure-observable-streams">eliminate global mutable state using “Pure” Observable Streams</a></li>
  <li>Then, we’ll <a href="#adding-physics-and-handling-more-inputs">add physics and handling more inputs</a></li>
  <li>We’ll <a href="#view">isolate the view</a></li>
  <li>Next, we’ll <a href="#additional-objects">introduce other objects, starting with bullets</a></li>
  <li>Finally, we’ll <a href="#collisions">deal with collisions</a></li>
</ul>

<p>Let’s start by making the svg with a simple polygon for the ship.  It will look like this:</p>

<p><img width="100" src="ship.png" /></p>

<p>And here’s the snippet of html that creates the ship:</p>
<div class="language-html highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nt">&lt;svg</span> <span class="na">width=</span><span class="s">"150"</span> <span class="na">height=</span><span class="s">"150"</span> <span class="na">style=</span><span class="s">"background-color:black"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;g</span> <span class="na">id=</span><span class="s">"ship"</span> <span class="na">transform=</span><span class="s">"translate(75,75)"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;polygon</span> <span class="na">points=</span><span class="s">"-15,20 15,20 0,-20"</span>
                <span class="na">style=</span><span class="s">"fill:lightblue"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;/polygon&gt;</span>
  <span class="nt">&lt;/g&gt;</span>
<span class="nt">&lt;/svg&gt;</span>
</code></pre></div></div>

<p>Note that the ship is rendered inside a transform group <code class="language-plaintext highlighter-rouge">&lt;g&gt;</code>.  We will be changing the <code class="language-plaintext highlighter-rouge">transform</code> attribute to move the ship around.</p>

<h2 id="rotating-the-ship">Rotating the ship</h2>
<p>To begin with we’ll make it possible for the player to rotate the ship with the arrow keys.  First, by directly adding listeners to keyboard events.  Then, by using events via Observable streams.  Here’s a preview of what it’s going to look like (<a href="https://stackblitz.com/edit/asteroids01">or you can play with it in a live editor</a>):</p>

<p><a href="https://stackblitz.com/edit/asteroids01"><img src="/asteroids/AsteroidsRotate.gif" alt="Rotation animation" /></a></p>

<p>There are basically just two states, as sketched in the following state machine:</p>

<p><img width="300" src="TurnStateMachine.png" /></p>

<h1 id="using-events-directly">Using Events Directly</h1>
<p>The first event we assign a function to is the window load event.  This function will not be invoked until the page is fully loaded, and therefore the SVG objects will be available.  Thus, our code begins:</p>

<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">window</span><span class="p">.</span><span class="nx">onload</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> <span class="nx">ship</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="dl">"</span><span class="s2">ship</span><span class="dl">"</span><span class="p">)</span><span class="o">!</span><span class="p">;</span>
  <span class="p">...</span>
</code></pre></div></div>
<p>So <code class="language-plaintext highlighter-rouge">ship</code> will reference the SVG <code class="language-plaintext highlighter-rouge">&lt;g&gt;</code> whose transform attribute we will be manipulating to move it.  To apply an incremental movement, such as rotating the ship by a certain angle relative to its current orientation, we will need to store that current location.  We could read it out of the transform attribute stored in the SVG, but that requires some messy string parsing.  We’ll just store the state in a local object, which we will keep up to date as we move the ship.  For now, all we have is the ship’s position (x and y coordinates) and rotation angle:</p>
<div class="language-javascript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">...</span>
  <span class="kd">const</span> <span class="nx">state</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">x</span><span class="p">:</span><span class="mi">100</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span><span class="mi">100</span><span class="p">,</span> <span class="na">angle</span><span class="p">:</span><span class="mi">0</span>
  <span class="p">}</span>
<span class="p">...</span>
</code></pre></div></div>
<p>Next, we need to specify a function to be invoked on keydown events:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">...</span>
  <span class="nb">document</span><span class="p">.</span><span class="nx">onkeydown</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">d</span><span class="p">:</span><span class="nx">KeyboardEvent</span><span class="p">)</span> <span class="p">{</span>
<span class="p">...</span>
</code></pre></div></div>
<p>Inside this function, we are only interested in left and right arrow keys.  If the keys are held down, after a moment they may start repeating automatically (this is OS dependent) and will churn out continuous keydown events.  We filter these out too by inspecting the KeyboardEvent.repeat property:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">...</span>
    <span class="k">if</span><span class="p">((</span><span class="nx">d</span><span class="p">.</span><span class="nx">key</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">ArrowLeft</span><span class="dl">"</span> <span class="o">||</span> <span class="nx">d</span><span class="p">.</span><span class="nx">key</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">ArrowRight</span><span class="dl">"</span><span class="p">)</span> <span class="o">&amp;&amp;</span> <span class="o">!</span><span class="nx">d</span><span class="p">.</span><span class="nx">repeat</span><span class="p">)</span> <span class="p">{</span>
<span class="p">...</span>
</code></pre></div></div>
<p>Let’s say we want a left- or right-arrow keydown event to start the ship rotating, and we want to keep rotating until the key is released.  To achieve this, we use the builtin <code class="language-plaintext highlighter-rouge">setInterval(f,i)</code> function, which invokes the function <code class="language-plaintext highlighter-rouge">f</code> repeatedly with the specified interval <code class="language-plaintext highlighter-rouge">i</code> delay (in milliseconds) between each invocation.  <code class="language-plaintext highlighter-rouge">setInterval</code> returns a numeric handle which we need to store so that we can clear the interval behaviour later.</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>      <span class="kd">const</span> <span class="nx">handle</span> <span class="o">=</span> <span class="nx">setInterval</span><span class="p">(</span><span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
        <span class="nx">ship</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">transform</span><span class="dl">'</span><span class="p">,</span>
          <span class="s2">`translate(</span><span class="p">${</span><span class="nx">state</span><span class="p">.</span><span class="nx">x</span><span class="p">}</span><span class="s2">,</span><span class="p">${</span><span class="nx">state</span><span class="p">.</span><span class="nx">y</span><span class="p">}</span><span class="s2">) rotate(</span><span class="p">${</span><span class="nx">state</span><span class="p">.</span><span class="nx">angle</span><span class="o">+=</span><span class="nx">d</span><span class="p">.</span><span class="nx">key</span> <span class="o">===</span> <span class="dl">"</span><span class="s2">ArrowLeft</span><span class="dl">"</span> <span class="p">?</span> <span class="o">-</span><span class="mi">1</span> <span class="p">:</span> <span class="mi">1</span><span class="p">}</span><span class="s2">)`</span><span class="p">)</span>
        <span class="p">},</span> <span class="mi">10</span><span class="p">);</span>
</code></pre></div></div>
<p>So as promised, this function is setting the <code class="language-plaintext highlighter-rouge">transform</code> property on the ship, using the position and angle information stored in our local <code class="language-plaintext highlighter-rouge">state</code> object.  We compute the new position by deducting or removing 1 (degree) from the angle (for a left or right rotation respectively) and simultaneously update the state object with the new angle.
Since we specify 10 milliseconds delay, the ship will rotate 100 times per second.</p>

<p>We’re not done yet.  We have to stop the rotation on keyup by calling <code class="language-plaintext highlighter-rouge">clearInterval</code>, for the specific interval we just created on keydown (using the <code class="language-plaintext highlighter-rouge">handle</code> we stored).  To do this, we’ll use <code class="language-plaintext highlighter-rouge">document.addEventListener</code> to specify a separate keyup handler for each keydown event, and since we will be creating a new keyup listener for each keydown event, we will also have to cleanup after ourselves or we’ll have a memory (event) leak:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">...</span>
      <span class="kd">const</span> <span class="nx">keyupListener</span> <span class="o">=</span> <span class="kd">function</span><span class="p">(</span><span class="nx">u</span><span class="p">:</span><span class="nx">KeyboardEvent</span><span class="p">)</span> <span class="p">{</span>
        <span class="k">if</span><span class="p">(</span><span class="nx">u</span><span class="p">.</span><span class="nx">key</span> <span class="o">===</span> <span class="nx">d</span><span class="p">.</span><span class="nx">key</span><span class="p">)</span> <span class="p">{</span>
          <span class="nx">clearInterval</span><span class="p">(</span><span class="nx">handle</span><span class="p">);</span>
          <span class="nb">document</span><span class="p">.</span><span class="nx">removeEventListener</span><span class="p">(</span><span class="dl">'</span><span class="s1">keyup</span><span class="dl">'</span><span class="p">,</span><span class="nx">keyupListener</span><span class="p">);</span>
        <span class="p">}</span>
      <span class="p">};</span>
      <span class="nb">document</span><span class="p">.</span><span class="nx">addEventListener</span><span class="p">(</span><span class="dl">"</span><span class="s2">keyup</span><span class="dl">"</span><span class="p">,</span><span class="nx">keyupListener</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre></div></div>
<p>And finally we’re done.  But it was surprisingly messy for what should be a relatively straightforward and commonplace interaction.  Furthermore, the imperative style code above finished up deeply nested with function declarations inside function declarations, inside <code class="language-plaintext highlighter-rouge">if</code>s and variables like <code class="language-plaintext highlighter-rouge">d</code>, <code class="language-plaintext highlighter-rouge">handle</code> and <code class="language-plaintext highlighter-rouge">keyupListener</code> are referenced from inside these nested function scopes in ways that are difficult to read and make sense of.  The state machine is relatively straightforward, but it’s tangled up by imperative code blocks.</p>

<h1 id="using-observable">Using Observable</h1>
<p>Observable (we’ll use the implementation from rxjs) wraps common asynchronous actions like user events and intervals in streams, that we can process with a chain of ‘operators’ applied to the chain through a <code class="language-plaintext highlighter-rouge">pipe</code>.</p>

<p>We start more or less the same as before, inside a function applied on <code class="language-plaintext highlighter-rouge">window.onload</code> and we still need local variables for the ship visual and its position/angle:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">window</span><span class="p">.</span><span class="nx">onload</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">const</span> 
    <span class="nx">ship</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="dl">"</span><span class="s2">ship</span><span class="dl">"</span><span class="p">)</span><span class="o">!</span><span class="p">,</span>
    <span class="nx">state</span> <span class="o">=</span> <span class="p">{</span> <span class="na">x</span><span class="p">:</span><span class="mi">100</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span><span class="mi">100</span><span class="p">,</span> <span class="na">angle</span><span class="p">:</span><span class="mi">0</span> <span class="p">};</span>
<span class="p">...</span>
</code></pre></div></div>
<p>But now we use the rxjs <code class="language-plaintext highlighter-rouge">fromEvent</code> function to create an Observable <code class="language-plaintext highlighter-rouge">keydown$</code> (the ‘$’ is a convention indicating the variable is a stream),</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> <span class="nx">keydown$</span> <span class="o">=</span> <span class="nx">fromEvent</span><span class="o">&lt;</span><span class="nx">KeyboardEvent</span><span class="o">&gt;</span><span class="p">(</span><span class="nb">document</span><span class="p">,</span> <span class="dl">'</span><span class="s1">keydown</span><span class="dl">'</span><span class="p">);</span>
</code></pre></div></div>
<p>The objects coming through the stream are of type <code class="language-plaintext highlighter-rouge">KeyboardEvent</code>, meaning they have the <code class="language-plaintext highlighter-rouge">key</code> and <code class="language-plaintext highlighter-rouge">repeat</code> properties we used before.  We can create a new stream which filters these out:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> <span class="nx">arrowKeys$</span> <span class="o">=</span> <span class="nx">keydown$</span><span class="p">.</span><span class="nx">pipe</span><span class="p">(</span>
    <span class="nx">filter</span><span class="p">(({</span><span class="nx">key</span><span class="p">})</span><span class="o">=&gt;</span><span class="nx">key</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">ArrowLeft</span><span class="dl">'</span> <span class="o">||</span> <span class="nx">key</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">ArrowRight</span><span class="dl">'</span><span class="p">),</span>
    <span class="nx">filter</span><span class="p">(({</span><span class="nx">repeat</span><span class="p">})</span><span class="o">=&gt;!</span><span class="nx">repeat</span><span class="p">));</span>
</code></pre></div></div>
<p>To duplicate the behaviour of our event driven version we need to rotate every 10ms.  We can make a stream which fires every 10ms using <code class="language-plaintext highlighter-rouge">interval(10)</code>, which we can “graft” onto our <code class="language-plaintext highlighter-rouge">arrowKeys$</code> stream using <code class="language-plaintext highlighter-rouge">mergeMap</code>.  We use <code class="language-plaintext highlighter-rouge">takeUntil</code> to terminate the interval on a <code class="language-plaintext highlighter-rouge">'keyup'</code>, filtered to ignore keys other than the one that initiated the <code class="language-plaintext highlighter-rouge">'keydown'</code>.  At the end of the <code class="language-plaintext highlighter-rouge">mergeMap</code> <code class="language-plaintext highlighter-rouge">pipe</code> we use <code class="language-plaintext highlighter-rouge">map</code> to return <code class="language-plaintext highlighter-rouge">d</code>, the original keydown <code class="language-plaintext highlighter-rouge">KeyboardEvent</code> object.  Back at the top-level <code class="language-plaintext highlighter-rouge">pipe</code> on arrowKeys$ we inspect this <code class="language-plaintext highlighter-rouge">KeyboardEvent</code> object to see whether we need a left or right rotation (positive or negative angle).  Thus, <code class="language-plaintext highlighter-rouge">angle$</code> is just a stream of <code class="language-plaintext highlighter-rouge">-1</code> and <code class="language-plaintext highlighter-rouge">1</code>.</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> <span class="nx">angle$</span> <span class="o">=</span> <span class="nx">arrowKeys$</span><span class="p">.</span><span class="nx">pipe</span><span class="p">(</span>
    <span class="nx">mergeMap</span><span class="p">(</span><span class="nx">d</span><span class="o">=&gt;</span><span class="nx">interval</span><span class="p">(</span><span class="mi">10</span><span class="p">).</span><span class="nx">pipe</span><span class="p">(</span>
      <span class="nx">takeUntil</span><span class="p">(</span><span class="nx">fromEvent</span><span class="o">&lt;</span><span class="nx">KeyboardEvent</span><span class="o">&gt;</span><span class="p">(</span><span class="nb">document</span><span class="p">,</span> <span class="dl">'</span><span class="s1">keyup</span><span class="dl">'</span><span class="p">).</span><span class="nx">pipe</span><span class="p">(</span>
        <span class="nx">filter</span><span class="p">(({</span><span class="nx">key</span><span class="p">})</span><span class="o">=&gt;</span><span class="nx">key</span> <span class="o">===</span> <span class="nx">d</span><span class="p">.</span><span class="nx">key</span><span class="p">)</span>
      <span class="p">)),</span>
      <span class="nx">map</span><span class="p">(</span><span class="nx">_</span><span class="o">=&gt;</span><span class="nx">d</span><span class="p">))</span>
    <span class="p">),</span>
    <span class="nx">map</span><span class="p">(</span><span class="nx">d</span><span class="o">=&gt;</span><span class="nx">d</span><span class="p">.</span><span class="nx">key</span><span class="o">===</span><span class="dl">'</span><span class="s1">ArrowLeft</span><span class="dl">'</span><span class="p">?</span><span class="o">-</span><span class="mi">1</span><span class="p">:</span><span class="mi">1</span><span class="p">));</span>
</code></pre></div></div>
<p>Finally, we <code class="language-plaintext highlighter-rouge">subscribe</code> to the <code class="language-plaintext highlighter-rouge">angle$</code> stream to perform our effectful code, updating <code class="language-plaintext highlighter-rouge">state</code> and rotating <code class="language-plaintext highlighter-rouge">ship</code>.</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="nx">angle$</span><span class="p">.</span><span class="nx">subscribe</span><span class="p">(</span><span class="nx">a</span><span class="o">=&gt;</span>
      <span class="nx">ship</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">transform</span><span class="dl">'</span><span class="p">,</span>
       <span class="s2">`translate(</span><span class="p">${</span><span class="nx">state</span><span class="p">.</span><span class="nx">x</span><span class="p">}</span><span class="s2">,</span><span class="p">${</span><span class="nx">state</span><span class="p">.</span><span class="nx">y</span><span class="p">}</span><span class="s2">) rotate(</span><span class="p">${</span><span class="nx">state</span><span class="p">.</span><span class="nx">angle</span><span class="o">+=</span><span class="nx">a</span><span class="p">}</span><span class="s2">)`</span><span class="p">)</span>
  <span class="p">)</span>
<span class="p">}</span>
</code></pre></div></div>
<p>Arguably, the Observable code has many advantages over the event handling code:</p>
<ul>
  <li>the streams created by <code class="language-plaintext highlighter-rouge">fromEvent</code> and <code class="language-plaintext highlighter-rouge">interval</code> automatically clean up the underlying events and interval handles when the streams complete.</li>
  <li>the ‘stream’ abstraction provided by observable gives us an intuitive way to think about asynchronous behaviour and chain transformations of the stream together through <code class="language-plaintext highlighter-rouge">pipe</code>s.</li>
  <li>We didn’t see it so much here, but the various observable streams we created are composable, in the sense that adding new pipes (or potentially multiple <code class="language-plaintext highlighter-rouge">subscribe</code>s) to them allow us to reuse and plug them together in powerful ways.</li>
</ul>

<h3 id="pure-observable-streams">Pure Observable Streams</h3>

<p>A weakness of the above implementation using Observable streams, is that we still have global mutable state.  Deep in the function passed to subscribe we alter the angle attribute on the <code class="language-plaintext highlighter-rouge">state</code> object.  Another Observable operator <code class="language-plaintext highlighter-rouge">scan</code>, allows us to capture this state transformation inside the stream, using a pure function to transform the state, i.e. a function that takes an input state object and—rather than altering it in-place—creates a new output state object with whatever change is required.</p>

<p>We’ll start by altering the start of our code to define an <code class="language-plaintext highlighter-rouge">interface</code> for <code class="language-plaintext highlighter-rouge">State</code> with <code class="language-plaintext highlighter-rouge">readonly</code> members, and we’ll place our <code class="language-plaintext highlighter-rouge">initialState</code> in a <code class="language-plaintext highlighter-rouge">const</code> variable that matches this interface.  You can also <a href="https://stackblitz.com/edit/asteroids02">play with the code in a live editor</a>.</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">window</span><span class="p">.</span><span class="nx">onload</span> <span class="o">=</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
  <span class="kd">type</span> <span class="nx">State</span> <span class="o">=</span> <span class="nb">Readonly</span><span class="o">&lt;</span><span class="p">{</span>
    <span class="na">x</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
    <span class="nl">y</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
    <span class="nl">angle</span><span class="p">:</span> <span class="kr">number</span><span class="p">;</span>
  <span class="p">}</span><span class="o">&gt;</span>
  <span class="kd">const</span> <span class="nx">initialState</span><span class="p">:</span> <span class="nx">State</span> <span class="o">=</span> <span class="p">{</span> <span class="na">x</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span> <span class="na">y</span><span class="p">:</span> <span class="mi">100</span><span class="p">,</span> <span class="na">angle</span><span class="p">:</span> <span class="mi">0</span><span class="p">};</span>
</code></pre></div></div>
<p>Now we’ll create a function that is a pure transformation of <code class="language-plaintext highlighter-rouge">State</code>:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">...</span>
  <span class="kd">function</span> <span class="nx">rotate</span><span class="p">(</span><span class="nx">s</span><span class="p">:</span><span class="nx">State</span><span class="p">,</span> <span class="nx">angleDelta</span><span class="p">:</span><span class="kr">number</span><span class="p">):</span> <span class="nx">State</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span> <span class="p">...</span><span class="nx">s</span><span class="p">,</span> <span class="c1">// copies the members of the input state for all but:</span>
      <span class="na">angle</span><span class="p">:</span> <span class="nx">s</span><span class="p">.</span><span class="nx">angle</span> <span class="o">+</span> <span class="nx">angleDelta</span>  <span class="c1">// only the angle is different in the new State</span>
    <span class="p">}</span>
  <span class="p">}</span>
<span class="p">...</span>
</code></pre></div></div>
<p>Next, we have another, completely self contained function to update the SVG for a given state:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">...</span>
  <span class="kd">function</span> <span class="nx">updateView</span><span class="p">(</span><span class="nx">state</span><span class="p">:</span><span class="nx">State</span><span class="p">):</span> <span class="k">void</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">ship</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="dl">"</span><span class="s2">ship</span><span class="dl">"</span><span class="p">)</span><span class="o">!</span><span class="p">;</span>
    <span class="nx">ship</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">transform</span><span class="dl">'</span><span class="p">,</span>
     <span class="s2">`translate(</span><span class="p">${</span><span class="nx">state</span><span class="p">.</span><span class="nx">x</span><span class="p">}</span><span class="s2">,</span><span class="p">${</span><span class="nx">state</span><span class="p">.</span><span class="nx">y</span><span class="p">}</span><span class="s2">) rotate(</span><span class="p">${</span><span class="nx">state</span><span class="p">.</span><span class="nx">angle</span><span class="p">}</span><span class="s2">)`</span><span class="p">)</span>
  <span class="p">}</span>
<span class="p">...</span>
</code></pre></div></div>
<p>And now our main <code class="language-plaintext highlighter-rouge">pipe</code> (collapsed into one) ends with a <code class="language-plaintext highlighter-rouge">scan</code> which “transduces” (transforms and reduces) our state, and the <code class="language-plaintext highlighter-rouge">subscribe</code> is a trivial call to <code class="language-plaintext highlighter-rouge">updateView</code>:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">...</span>
  <span class="nx">fromEvent</span><span class="o">&lt;</span><span class="nx">KeyboardEvent</span><span class="o">&gt;</span><span class="p">(</span><span class="nb">document</span><span class="p">,</span> <span class="dl">'</span><span class="s1">keydown</span><span class="dl">'</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">pipe</span><span class="p">(</span>
      <span class="nx">filter</span><span class="p">(({</span><span class="nx">code</span><span class="p">})</span><span class="o">=&gt;</span><span class="nx">code</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">ArrowLeft</span><span class="dl">'</span> <span class="o">||</span> <span class="nx">code</span> <span class="o">===</span> <span class="dl">'</span><span class="s1">ArrowRight</span><span class="dl">'</span><span class="p">),</span>
      <span class="nx">filter</span><span class="p">(({</span><span class="nx">repeat</span><span class="p">})</span><span class="o">=&gt;!</span><span class="nx">repeat</span><span class="p">),</span>
      <span class="nx">mergeMap</span><span class="p">(</span><span class="nx">d</span><span class="o">=&gt;</span><span class="nx">interval</span><span class="p">(</span><span class="mi">10</span><span class="p">).</span><span class="nx">pipe</span><span class="p">(</span>
        <span class="nx">takeUntil</span><span class="p">(</span><span class="nx">fromEvent</span><span class="o">&lt;</span><span class="nx">KeyboardEvent</span><span class="o">&gt;</span><span class="p">(</span><span class="nb">document</span><span class="p">,</span> <span class="dl">'</span><span class="s1">keyup</span><span class="dl">'</span><span class="p">).</span><span class="nx">pipe</span><span class="p">(</span>
          <span class="nx">filter</span><span class="p">(({</span><span class="nx">code</span><span class="p">})</span><span class="o">=&gt;</span><span class="nx">code</span> <span class="o">===</span> <span class="nx">d</span><span class="p">.</span><span class="nx">code</span><span class="p">)</span>
        <span class="p">)),</span>
        <span class="nx">map</span><span class="p">(</span><span class="nx">_</span><span class="o">=&gt;</span><span class="nx">d</span><span class="p">))</span>
      <span class="p">),</span>
      <span class="nx">map</span><span class="p">(({</span><span class="nx">code</span><span class="p">})</span><span class="o">=&gt;</span><span class="nx">code</span><span class="o">===</span><span class="dl">'</span><span class="s1">ArrowLeft</span><span class="dl">'</span><span class="p">?</span><span class="o">-</span><span class="mi">1</span><span class="p">:</span><span class="mi">1</span><span class="p">),</span>
      <span class="nx">scan</span><span class="p">(</span><span class="nx">rotate</span><span class="p">,</span> <span class="nx">initialState</span><span class="p">))</span>
    <span class="p">.</span><span class="nx">subscribe</span><span class="p">(</span><span class="nx">updateView</span><span class="p">)</span>
<span class="p">}</span>
</code></pre></div></div>
<p>The code above is a bit longer than what we started with, but it’s starting to lay a more extensible framework for a more complete game.  And it has some nice architectural properties, in particular we’ve completely decoupled our view code from our state management.  We could swap out SVG for a completely different UI by replacing the updateView function.</p>

<h2 id="adding-physics-and-handling-more-inputs">Adding Physics and Handling More Inputs</h2>
<p>Classic Asteroids is more of a space flight simulator in a weird toroidal topology than the ‘static’ rotation that we’ve provided above.  We will make our spaceship a freely floating body in space, with directional and rotational velocity.
We are going to need more inputs than just left and right arrow keys to pilot our ship too.</p>

<p>Here’s a sneak preview of what this next stage will look like (click on the image to try it out in a live code editor):</p>

<p><a href="https://stackblitz.com/edit/asteroids03?file=index.ts"><img src="AsteroidsFly.gif" alt="Spaceship flying" /></a></p>

<h1 id="input-actions">Input Actions</h1>
<p>Let’s start with adding “thrust” in response to up arrow.
With the code above, adding more and more intervals triggered by key down events would get increasingly messy.
Rather than have streams triggered by key events, for the purposes of simulation it makes sense that our main Observable be a stream of discrete timesteps.  Thus, we’ll be moving our <code class="language-plaintext highlighter-rouge">interval(10)</code> to the top of our pipe.</p>

<p>Before we had just left and right rotations coming out of our stream.  Our new stream is going to have multiple types of actions as payload:</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">Tick</code> - a discrete timestep in our simulation, triggered by <code class="language-plaintext highlighter-rouge">interval</code>.</li>
  <li><code class="language-plaintext highlighter-rouge">Rotate</code> - a ship rotation triggered by left or right arrows keys</li>
  <li><code class="language-plaintext highlighter-rouge">Thrust</code> - fire the boosters! Using the up-arrow key</li>
</ul>

<p>Each of these actions has some data, we’ll model them as simple classes (note that we need classes rather than anonymous interfaces here because the <code class="language-plaintext highlighter-rouge">instanceof</code> pattern-match used inside the <code class="language-plaintext highlighter-rouge">reduceState</code> function <a href="#reducing-state">below</a> requires the presence of a constructor):</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">class</span> <span class="nx">Tick</span> <span class="p">{</span> <span class="kd">constructor</span><span class="p">(</span><span class="k">public</span> <span class="k">readonly</span> <span class="nx">elapsed</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span> <span class="p">{}</span> <span class="p">}</span>
<span class="kd">class</span> <span class="nx">Rotate</span> <span class="p">{</span> <span class="kd">constructor</span><span class="p">(</span><span class="k">public</span> <span class="k">readonly</span> <span class="nx">direction</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span> <span class="p">{}</span> <span class="p">}</span>
<span class="kd">class</span> <span class="nx">Thrust</span> <span class="p">{</span> <span class="kd">constructor</span><span class="p">(</span><span class="k">public</span> <span class="k">readonly</span> <span class="nx">on</span><span class="p">:</span><span class="nx">boolean</span><span class="p">)</span> <span class="p">{}</span> <span class="p">}</span>
</code></pre></div></div>

<p>Now, we’ll create separate Observables for each of the key events.  There’s a repetitive pattern in creating each of these Observables, for a given:</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">Event</code> - keydown or keyup</li>
  <li><code class="language-plaintext highlighter-rouge">Key</code> - one of the arrows (for now)</li>
  <li>produce an Observable stream of a particular action type.</li>
</ul>

<p>It sounds like something we can model with a nice reusable function:</p>

<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">type</span> <span class="nx">Event</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">keydown</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">keyup</span><span class="dl">'</span>
  <span class="kd">type</span> <span class="nx">Key</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">ArrowLeft</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">ArrowRight</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">ArrowUp</span><span class="dl">'</span>
  <span class="kd">const</span> <span class="nx">observeKey</span> <span class="o">=</span> <span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span><span class="nx">eventName</span><span class="p">:</span><span class="kr">string</span><span class="p">,</span> <span class="nx">k</span><span class="p">:</span><span class="nx">Key</span><span class="p">,</span> <span class="nx">result</span><span class="p">:()</span><span class="o">=&gt;</span><span class="nx">T</span><span class="p">)</span><span class="o">=&gt;</span>
    <span class="nx">fromEvent</span><span class="o">&lt;</span><span class="nx">KeyboardEvent</span><span class="o">&gt;</span><span class="p">(</span><span class="nb">document</span><span class="p">,</span><span class="nx">eventName</span><span class="p">)</span>
      <span class="p">.</span><span class="nx">pipe</span><span class="p">(</span>
        <span class="nx">filter</span><span class="p">(({</span><span class="nx">code</span><span class="p">})</span><span class="o">=&gt;</span><span class="nx">code</span> <span class="o">===</span> <span class="nx">k</span><span class="p">),</span>
        <span class="nx">filter</span><span class="p">(({</span><span class="nx">repeat</span><span class="p">})</span><span class="o">=&gt;!</span><span class="nx">repeat</span><span class="p">),</span>
        <span class="nx">map</span><span class="p">(</span><span class="nx">result</span><span class="p">)),</span>
</code></pre></div></div>

<p>Now we have all the pieces to create a whole slew of input streams:</p>

<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span>
    <span class="nx">startLeftRotate</span> <span class="o">=</span> <span class="nx">observeKey</span><span class="p">(</span><span class="dl">'</span><span class="s1">keydown</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">ArrowLeft</span><span class="dl">'</span><span class="p">,()</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Rotate</span><span class="p">(</span><span class="o">-</span><span class="p">.</span><span class="mi">1</span><span class="p">)),</span>
    <span class="nx">startRightRotate</span> <span class="o">=</span> <span class="nx">observeKey</span><span class="p">(</span><span class="dl">'</span><span class="s1">keydown</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">ArrowRight</span><span class="dl">'</span><span class="p">,()</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Rotate</span><span class="p">(.</span><span class="mi">1</span><span class="p">)),</span>
    <span class="nx">stopLeftRotate</span> <span class="o">=</span> <span class="nx">observeKey</span><span class="p">(</span><span class="dl">'</span><span class="s1">keyup</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">ArrowLeft</span><span class="dl">'</span><span class="p">,()</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Rotate</span><span class="p">(</span><span class="mi">0</span><span class="p">)),</span>
    <span class="nx">stopRightRotate</span> <span class="o">=</span> <span class="nx">observeKey</span><span class="p">(</span><span class="dl">'</span><span class="s1">keyup</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">ArrowRight</span><span class="dl">'</span><span class="p">,()</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Rotate</span><span class="p">(</span><span class="mi">0</span><span class="p">)),</span>
    <span class="nx">startThrust</span> <span class="o">=</span> <span class="nx">observeKey</span><span class="p">(</span><span class="dl">'</span><span class="s1">keydown</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">ArrowUp</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Thrust</span><span class="p">(</span><span class="kc">true</span><span class="p">)),</span>
    <span class="nx">stopThrust</span> <span class="o">=</span> <span class="nx">observeKey</span><span class="p">(</span><span class="dl">'</span><span class="s1">keyup</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">ArrowUp</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Thrust</span><span class="p">(</span><span class="kc">false</span><span class="p">))</span>
</code></pre></div></div>

<h1 id="vector-maths">Vector Maths</h1>

<p>Since we’re going to start worrying about the physics of our simulation, we’re going to need some helper code. First, a handy dandy Vector class.  It’s just standard vector maths, so hopefully self explanatory.  In the spirit of being pure and declarative I’ve made it immutable, and all the functions (except <code class="language-plaintext highlighter-rouge">len</code> which returns a number) return new instances of <code class="language-plaintext highlighter-rouge">Vec</code> rather than changing its data in place.</p>

<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">class</span> <span class="nx">Vec</span> <span class="p">{</span>
  <span class="kd">constructor</span><span class="p">(</span><span class="k">public</span> <span class="k">readonly</span> <span class="nx">x</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">0</span><span class="p">,</span> <span class="k">public</span> <span class="k">readonly</span> <span class="nx">y</span><span class="p">:</span> <span class="kr">number</span> <span class="o">=</span> <span class="mi">0</span><span class="p">)</span> <span class="p">{}</span>
  <span class="nx">add</span> <span class="o">=</span> <span class="p">(</span><span class="nx">b</span><span class="p">:</span><span class="nx">Vec</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">new</span> <span class="nx">Vec</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">x</span> <span class="o">+</span> <span class="nx">b</span><span class="p">.</span><span class="nx">x</span><span class="p">,</span> <span class="k">this</span><span class="p">.</span><span class="nx">y</span> <span class="o">+</span> <span class="nx">b</span><span class="p">.</span><span class="nx">y</span><span class="p">)</span>
  <span class="nx">sub</span> <span class="o">=</span> <span class="p">(</span><span class="nx">b</span><span class="p">:</span><span class="nx">Vec</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">this</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="nx">b</span><span class="p">.</span><span class="nx">scale</span><span class="p">(</span><span class="o">-</span><span class="mi">1</span><span class="p">))</span>
  <span class="nx">len</span> <span class="o">=</span> <span class="p">()</span><span class="o">=&gt;</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">sqrt</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">x</span><span class="o">*</span><span class="k">this</span><span class="p">.</span><span class="nx">x</span> <span class="o">+</span> <span class="k">this</span><span class="p">.</span><span class="nx">y</span><span class="o">*</span><span class="k">this</span><span class="p">.</span><span class="nx">y</span><span class="p">)</span>
  <span class="nx">scale</span> <span class="o">=</span> <span class="p">(</span><span class="nx">s</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">new</span> <span class="nx">Vec</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">x</span><span class="o">*</span><span class="nx">s</span><span class="p">,</span><span class="k">this</span><span class="p">.</span><span class="nx">y</span><span class="o">*</span><span class="nx">s</span><span class="p">)</span>
  <span class="nx">ortho</span> <span class="o">=</span> <span class="p">()</span><span class="o">=&gt;</span> <span class="k">new</span> <span class="nx">Vec</span><span class="p">(</span><span class="k">this</span><span class="p">.</span><span class="nx">y</span><span class="p">,</span><span class="o">-</span><span class="k">this</span><span class="p">.</span><span class="nx">x</span><span class="p">)</span>
  <span class="nx">rotate</span> <span class="o">=</span> <span class="p">(</span><span class="nx">deg</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span> <span class="o">=&gt;</span>
            <span class="p">(</span><span class="nx">rad</span> <span class="o">=&gt;</span><span class="p">(</span>
                <span class="p">(</span><span class="nx">cos</span><span class="p">,</span><span class="nx">sin</span><span class="p">,{</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">})</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Vec</span><span class="p">(</span><span class="nx">x</span><span class="o">*</span><span class="nx">cos</span> <span class="o">-</span> <span class="nx">y</span><span class="o">*</span><span class="nx">sin</span><span class="p">,</span> <span class="nx">x</span><span class="o">*</span><span class="nx">sin</span> <span class="o">+</span> <span class="nx">y</span><span class="o">*</span><span class="nx">cos</span><span class="p">)</span>
              <span class="p">)(</span><span class="nb">Math</span><span class="p">.</span><span class="nx">cos</span><span class="p">(</span><span class="nx">rad</span><span class="p">),</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">sin</span><span class="p">(</span><span class="nx">rad</span><span class="p">),</span> <span class="k">this</span><span class="p">)</span>
            <span class="p">)(</span><span class="nb">Math</span><span class="p">.</span><span class="nx">PI</span> <span class="o">*</span> <span class="nx">deg</span> <span class="o">/</span> <span class="mi">180</span><span class="p">)</span>

  <span class="k">static</span> <span class="nx">unitVecInDirection</span> <span class="o">=</span> <span class="p">(</span><span class="nx">deg</span><span class="p">:</span> <span class="kr">number</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="k">new</span> <span class="nx">Vec</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span><span class="o">-</span><span class="mi">1</span><span class="p">).</span><span class="nx">rotate</span><span class="p">(</span><span class="nx">deg</span><span class="p">)</span>
  <span class="k">static</span> <span class="nx">Zero</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Vec</span><span class="p">();</span>
<span class="p">}</span>
</code></pre></div></div>
<p>To implement the toroidal topology of space, we’ll need to know the canvas size. 
For now, we’ll hard code it in a constant <code class="language-plaintext highlighter-rouge">CanvasSize</code>.  Alternately, we could query it from the svg element, or we could set the SVG size - maybe later.
The torus wrapping function will use the <code class="language-plaintext highlighter-rouge">CanvasSize</code> to determine the bounds and simply teleport any <code class="language-plaintext highlighter-rouge">Vec</code> which goes out of bounds to the opposite side.</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="kd">const</span> 
  <span class="nx">CanvasSize</span> <span class="o">=</span> <span class="mi">200</span><span class="p">,</span>
  <span class="nx">torusWrap</span> <span class="o">=</span> <span class="p">({</span><span class="nx">x</span><span class="p">,</span><span class="nx">y</span><span class="p">}:</span><span class="nx">Vec</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span> 
    <span class="kd">const</span> <span class="nx">wrap</span> <span class="o">=</span> <span class="p">(</span><span class="na">v</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span> <span class="o">=&gt;</span> 
      <span class="nx">v</span> <span class="o">&lt;</span> <span class="mi">0</span> <span class="p">?</span> <span class="nx">v</span> <span class="o">+</span> <span class="nx">CanvasSize</span> <span class="p">:</span> <span class="nx">v</span> <span class="o">&gt;</span> <span class="nx">CanvasSize</span> <span class="p">?</span> <span class="nx">v</span> <span class="o">-</span> <span class="nx">CanvasSize</span> <span class="p">:</span> <span class="nx">v</span><span class="p">;</span>
    <span class="k">return</span> <span class="k">new</span> <span class="nx">Vec</span><span class="p">(</span><span class="nx">wrap</span><span class="p">(</span><span class="nx">x</span><span class="p">),</span><span class="nx">wrap</span><span class="p">(</span><span class="nx">y</span><span class="p">))</span>
  <span class="p">};</span>
</code></pre></div></div>
<p>We’ll use <code class="language-plaintext highlighter-rouge">Vec</code> in a slightly richer set of State.</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">type</span> <span class="nx">State</span> <span class="o">=</span> <span class="nb">Readonly</span><span class="o">&lt;</span><span class="p">{</span>
    <span class="na">pos</span><span class="p">:</span><span class="nx">Vec</span><span class="p">,</span> 
    <span class="na">vel</span><span class="p">:</span><span class="nx">Vec</span><span class="p">,</span>
    <span class="na">acc</span><span class="p">:</span><span class="nx">Vec</span><span class="p">,</span>
    <span class="na">angle</span><span class="p">:</span><span class="kr">number</span><span class="p">,</span>
    <span class="na">rotation</span><span class="p">:</span><span class="kr">number</span><span class="p">,</span>
    <span class="na">torque</span><span class="p">:</span><span class="kr">number</span>
  <span class="p">}</span><span class="o">&gt;</span>
</code></pre></div></div>
<p>We create an <code class="language-plaintext highlighter-rouge">initialState</code> using <code class="language-plaintext highlighter-rouge">CanvasSize</code> to start the spaceship at the centre:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> <span class="nx">initialState</span><span class="p">:</span><span class="nx">State</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">pos</span><span class="p">:</span> <span class="k">new</span> <span class="nx">Vec</span><span class="p">(</span><span class="nx">CanvasSize</span><span class="o">/</span><span class="mi">2</span><span class="p">,</span><span class="nx">CanvasSize</span><span class="o">/</span><span class="mi">2</span><span class="p">),</span> 
      <span class="na">vel</span><span class="p">:</span> <span class="nx">Vec</span><span class="p">.</span><span class="nx">Zero</span><span class="p">,</span>
      <span class="na">acc</span><span class="p">:</span> <span class="nx">Vec</span><span class="p">.</span><span class="nx">Zero</span><span class="p">,</span>
      <span class="na">angle</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">rotation</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">torque</span><span class="p">:</span><span class="mi">0</span>
  <span class="p">}</span>
</code></pre></div></div>

<h1 id="reducing-state">Reducing State</h1>

<p>We can encapsulate all the possible transformations of state in a function:</p>

<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> <span class="nx">reduceState</span> <span class="o">=</span> <span class="p">(</span><span class="nx">s</span><span class="p">:</span><span class="nx">State</span><span class="p">,</span> <span class="nx">e</span><span class="p">:</span><span class="nx">Rotate</span><span class="o">|</span><span class="nx">Thrust</span><span class="o">|</span><span class="nx">Tick</span><span class="p">)</span><span class="o">=&gt;</span>
    <span class="nx">e</span> <span class="k">instanceof</span> <span class="nx">Rotate</span> <span class="p">?</span> <span class="p">{...</span><span class="nx">s</span><span class="p">,</span>
      <span class="na">torque</span><span class="p">:</span> <span class="nx">e</span><span class="p">.</span><span class="nx">direction</span>
    <span class="p">}</span> <span class="p">:</span>
    <span class="nx">e</span> <span class="k">instanceof</span> <span class="nx">Thrust</span> <span class="p">?</span> <span class="p">{...</span><span class="nx">s</span><span class="p">,</span>
      <span class="na">acc</span><span class="p">:</span><span class="nx">e</span><span class="p">.</span><span class="nx">on</span> <span class="p">?</span> <span class="nx">Vec</span><span class="p">.</span><span class="nx">unitVecInDirection</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">angle</span><span class="p">).</span><span class="nx">scale</span><span class="p">(</span><span class="mf">0.05</span><span class="p">)</span>
               <span class="p">:</span> <span class="nx">Vec</span><span class="p">.</span><span class="nx">Zero</span>
    <span class="p">}</span> <span class="p">:</span> <span class="p">{...</span><span class="nx">s</span><span class="p">,</span>
      <span class="na">rotation</span><span class="p">:</span> <span class="nx">s</span><span class="p">.</span><span class="nx">rotation</span><span class="o">+</span><span class="nx">s</span><span class="p">.</span><span class="nx">torque</span><span class="p">,</span>
      <span class="na">angle</span><span class="p">:</span> <span class="nx">s</span><span class="p">.</span><span class="nx">angle</span><span class="o">+</span><span class="nx">s</span><span class="p">.</span><span class="nx">rotation</span><span class="p">,</span>
      <span class="na">pos</span><span class="p">:</span> <span class="nx">torusWrap</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">pos</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">vel</span><span class="p">)),</span>
      <span class="na">vel</span><span class="p">:</span> <span class="nx">s</span><span class="p">.</span><span class="nx">vel</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">acc</span><span class="p">)</span>
    <span class="p">};</span>
</code></pre></div></div>

<p>And finally we <code class="language-plaintext highlighter-rouge">merge</code> our different inputs and scan over <code class="language-plaintext highlighter-rouge">State</code>, and the final <code class="language-plaintext highlighter-rouge">subscribe</code> calls the <code class="language-plaintext highlighter-rouge">updateView</code>, once again, a self-contained function which does whatever is required to render the State.  We describe the updated <code class="language-plaintext highlighter-rouge">updateView</code> in the next section.</p>

<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="nx">interval</span><span class="p">(</span><span class="mi">10</span><span class="p">)</span>
    <span class="p">.</span><span class="nx">pipe</span><span class="p">(</span>
      <span class="nx">map</span><span class="p">(</span><span class="nx">elapsed</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Tick</span><span class="p">(</span><span class="nx">elapsed</span><span class="p">)),</span>
      <span class="nx">merge</span><span class="p">(</span>
        <span class="nx">startLeftRotate</span><span class="p">,</span><span class="nx">startRightRotate</span><span class="p">,</span><span class="nx">stopLeftRotate</span><span class="p">,</span><span class="nx">stopRightRotate</span><span class="p">),</span>
      <span class="nx">merge</span><span class="p">(</span><span class="nx">startThrust</span><span class="p">,</span><span class="nx">stopThrust</span><span class="p">),</span>
      <span class="nx">scan</span><span class="p">(</span><span class="nx">reduceState</span><span class="p">,</span> <span class="nx">initialState</span><span class="p">))</span>
    <span class="p">.</span><span class="nx">subscribe</span><span class="p">(</span><span class="nx">updateView</span><span class="p">);</span>
</code></pre></div></div>

<p>Note, there are two versions of <code class="language-plaintext highlighter-rouge">merge</code> in rxjs. One is a function which merges multiple Observables and returns a new Observable, it is imported from <code class="language-plaintext highlighter-rouge">'rxjs'</code>.  Here we are using the operator version of <code class="language-plaintext highlighter-rouge">merge</code> to merge additional Observables into the pipe, imported from <code class="language-plaintext highlighter-rouge">'rxjs/operators'</code>.</p>

<h1 id="view">View</h1>

<p>Once again, the above completely decouples the view from state management.  But now we have a richer state, we have more stuff we can show in the view.  We’ll start with a little CSS, not only to style elements, but also to hide or show flame from our boosters.</p>
<div class="language-css highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nc">.ship</span> <span class="p">{</span>
  <span class="py">fill</span><span class="p">:</span> <span class="no">lightblue</span><span class="p">;</span>
<span class="p">}</span>
<span class="nc">.booster</span> <span class="p">{</span>
  <span class="py">fill</span><span class="p">:</span> <span class="no">orange</span><span class="p">;</span>
<span class="p">}</span>
<span class="nc">.hidden</span> <span class="p">{</span>
  <span class="nl">visibility</span><span class="p">:</span><span class="nb">hidden</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
<p>We add a few more polygons for the booster flames:</p>
<div class="language-html highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nt">&lt;script </span><span class="na">src=</span><span class="s">"/bundle.js"</span><span class="nt">&gt;&lt;/script&gt;</span>
<span class="nt">&lt;svg</span> <span class="na">id=</span><span class="s">"svgCanvas"</span> <span class="na">width=</span><span class="s">"200"</span> <span class="na">height=</span><span class="s">"200"</span> <span class="na">style=</span><span class="s">"background-color:black"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;g</span> <span class="na">id=</span><span class="s">"ship"</span> <span class="na">transform=</span><span class="s">"translate(100,100)"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;polygon</span> <span class="na">class=</span><span class="s">"booster hidden"</span> <span class="na">id=</span><span class="s">"forwardThrust"</span> <span class="na">points=</span><span class="s">"-3,20 0,35 3,20"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;/polygon&gt;</span>
    <span class="nt">&lt;polygon</span> <span class="na">class=</span><span class="s">"booster hidden"</span> <span class="na">id=</span><span class="s">"leftThrust"</span> <span class="na">points=</span><span class="s">"2,-10 15,-12 2,-14"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;/polygon&gt;</span>
    <span class="nt">&lt;polygon</span> <span class="na">class=</span><span class="s">"booster hidden"</span> <span class="na">id=</span><span class="s">"rightThrust"</span> <span class="na">points=</span><span class="s">"-2,-10 -15,-12 -2,-14"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;/polygon&gt;</span>
    <span class="nt">&lt;polygon</span> <span class="na">class=</span><span class="s">"ship"</span> <span class="na">points=</span><span class="s">"-15,20 15,20 0,-20"</span><span class="nt">&gt;</span>
    <span class="nt">&lt;/polygon&gt;</span>
  <span class="nt">&lt;/g&gt;</span>
<span class="nt">&lt;/svg&gt;</span>
</code></pre></div></div>
<p>And here’s our updated updateView function where we not only move the ship but also show flames shooting out of it as it powers around the torus:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">function</span> <span class="nx">updateView</span><span class="p">(</span><span class="nx">s</span><span class="p">:</span> <span class="nx">State</span><span class="p">)</span> <span class="p">{</span>
    <span class="kd">const</span> 
      <span class="nx">ship</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="dl">"</span><span class="s2">ship</span><span class="dl">"</span><span class="p">)</span><span class="o">!</span><span class="p">,</span>
      <span class="nx">show</span> <span class="o">=</span> <span class="p">(</span><span class="nx">id</span><span class="p">:</span><span class="kr">string</span><span class="p">,</span><span class="nx">condition</span><span class="p">:</span><span class="nx">boolean</span><span class="p">)</span><span class="o">=&gt;</span><span class="p">((</span><span class="nx">e</span><span class="p">:</span><span class="nx">HTMLElement</span><span class="p">)</span> <span class="o">=&gt;</span> 
        <span class="nx">condition</span> <span class="p">?</span> <span class="nx">e</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nx">remove</span><span class="p">(</span><span class="dl">'</span><span class="s1">hidden</span><span class="dl">'</span><span class="p">)</span>
                  <span class="p">:</span> <span class="nx">e</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="dl">'</span><span class="s1">hidden</span><span class="dl">'</span><span class="p">))(</span><span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="nx">id</span><span class="p">)</span><span class="o">!</span><span class="p">);</span>
    <span class="nx">show</span><span class="p">(</span><span class="dl">"</span><span class="s2">leftThrust</span><span class="dl">"</span><span class="p">,</span>  <span class="nx">s</span><span class="p">.</span><span class="nx">torque</span><span class="o">&lt;</span><span class="mi">0</span><span class="p">);</span>
    <span class="nx">show</span><span class="p">(</span><span class="dl">"</span><span class="s2">rightThrust</span><span class="dl">"</span><span class="p">,</span> <span class="nx">s</span><span class="p">.</span><span class="nx">torque</span><span class="o">&gt;</span><span class="mi">0</span><span class="p">);</span>
    <span class="nx">show</span><span class="p">(</span><span class="dl">"</span><span class="s2">forwardThrust</span><span class="dl">"</span><span class="p">,</span>    <span class="nx">s</span><span class="p">.</span><span class="nx">acc</span><span class="p">.</span><span class="nx">len</span><span class="p">()</span><span class="o">&gt;</span><span class="mi">0</span><span class="p">);</span> 
    <span class="nx">ship</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="dl">'</span><span class="s1">transform</span><span class="dl">'</span><span class="p">,</span> <span class="s2">`translate(</span><span class="p">${</span><span class="nx">s</span><span class="p">.</span><span class="nx">pos</span><span class="p">.</span><span class="nx">x</span><span class="p">}</span><span class="s2">,</span><span class="p">${</span><span class="nx">s</span><span class="p">.</span><span class="nx">pos</span><span class="p">.</span><span class="nx">y</span><span class="p">}</span><span class="s2">) rotate(</span><span class="p">${</span><span class="nx">s</span><span class="p">.</span><span class="nx">angle</span><span class="p">}</span><span class="s2">)`</span><span class="p">);</span>
  <span class="p">}</span>
</code></pre></div></div>

<h2 id="additional-objects">Additional Objects</h2>
<p>Things get more complicated when we start adding more objects to the canvas that all participate in the physics simulation.  Furthermore, objects like asteroids and bullets will need to be added and removed from the canvas dynamically - unlike the ship whose visual is currently defined in the <code class="language-plaintext highlighter-rouge">svg</code> and never leaves.</p>

<p>However, we now have all the pieces of our MVC architecture in place, all tied together with an observable stream:</p>

<p><img src="MVC.png" alt="Observable MVC Architecture" /></p>

<p>So completing the game is just a matter of:</p>
<ul>
  <li>adding more input actions (e.g. <code class="language-plaintext highlighter-rouge">Shoot</code>)</li>
  <li>extending our state data structure to include bullets, rocks, etc.</li>
  <li>extending our reduce state function to manipulate this State</li>
  <li>extending the <code class="language-plaintext highlighter-rouge">updateView</code> function so the user can see it</li>
</ul>

<p>We’ll start with bullets that can be fired with the Space key, and which expire after a set period of time:</p>

<p><a href="https://stackblitz.com/edit/asteroids04?file=index.ts"><img src="AsteroidsShoot.gif" alt="Spaceship flying" /></a></p>

<p>However, the basic framework above is a good basis on which to extend.</p>

<h1 id="per-object-state">Per-object State</h1>
<p>The first complication is generalising bodies that participate in the force model with their own type <code class="language-plaintext highlighter-rouge">Body</code>, separate from the <code class="language-plaintext highlighter-rouge">State</code>:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">type</span> <span class="nx">Body</span> <span class="o">=</span> <span class="nb">Readonly</span><span class="o">&lt;</span><span class="p">{</span>
    <span class="na">id</span><span class="p">:</span><span class="kr">string</span><span class="p">,</span>
    <span class="na">pos</span><span class="p">:</span><span class="nx">Vec</span><span class="p">,</span> 
    <span class="na">vel</span><span class="p">:</span><span class="nx">Vec</span><span class="p">,</span>
    <span class="na">thrust</span><span class="p">:</span><span class="nx">boolean</span><span class="p">,</span>
    <span class="na">angle</span><span class="p">:</span><span class="kr">number</span><span class="p">,</span>
    <span class="na">rotation</span><span class="p">:</span><span class="kr">number</span><span class="p">,</span>
    <span class="na">torque</span><span class="p">:</span><span class="kr">number</span><span class="p">,</span>
    <span class="na">radius</span><span class="p">:</span><span class="kr">number</span><span class="p">,</span>
    <span class="na">createTime</span><span class="p">:</span><span class="kr">number</span>
  <span class="p">}</span><span class="o">&gt;</span>
  <span class="kd">type</span> <span class="nx">State</span> <span class="o">=</span> <span class="nb">Readonly</span><span class="o">&lt;</span><span class="p">{</span>
    <span class="na">time</span><span class="p">:</span><span class="kr">number</span><span class="p">,</span>
    <span class="na">ship</span><span class="p">:</span><span class="nx">Body</span><span class="p">,</span>
    <span class="na">bullets</span><span class="p">:</span><span class="nx">ReadonlyArray</span><span class="o">&lt;</span><span class="nx">Body</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="na">exit</span><span class="p">:</span><span class="nx">ReadonlyArray</span><span class="o">&lt;</span><span class="nx">Body</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="na">objCount</span><span class="p">:</span><span class="kr">number</span>
  <span class="p">}</span><span class="o">&gt;</span>
</code></pre></div></div>
<p>So the <code class="language-plaintext highlighter-rouge">ship</code> is a <code class="language-plaintext highlighter-rouge">Body</code>, and we will have collections of <code class="language-plaintext highlighter-rouge">Body</code> for both <code class="language-plaintext highlighter-rouge">bullets</code> and <code class="language-plaintext highlighter-rouge">rocks</code>.  What’s this <code class="language-plaintext highlighter-rouge">exit</code> thing?  Well, when we remove something from the canvas, e.g. a bullet, we’ll create a new state with a copy of the <code class="language-plaintext highlighter-rouge">bullets</code> array minus the removed bullet, and we’ll add that removed bullet - together with any other removed <code class="language-plaintext highlighter-rouge">Body</code>s - to the <code class="language-plaintext highlighter-rouge">exit</code> array.  This notifies the <code class="language-plaintext highlighter-rouge">updateView</code> function that they can be removed.</p>

<p>Note the <code class="language-plaintext highlighter-rouge">objCount</code>.  This counter is incremented every time we add a <code class="language-plaintext highlighter-rouge">Body</code> and gives us a way to create a unique id that can be used to match the <code class="language-plaintext highlighter-rouge">Body</code> against its corresponding view object.</p>

<p>Now we define functions to create objects:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">function</span> <span class="nx">createBullet</span><span class="p">(</span><span class="nx">s</span><span class="p">:</span><span class="nx">State</span><span class="p">):</span><span class="nx">Body</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">d</span> <span class="o">=</span> <span class="nx">Vec</span><span class="p">.</span><span class="nx">unitVecInDirection</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">ship</span><span class="p">.</span><span class="nx">angle</span><span class="p">);</span>
    <span class="k">return</span> <span class="p">{</span>
      <span class="na">id</span><span class="p">:</span> <span class="s2">`bullet</span><span class="p">${</span><span class="nx">s</span><span class="p">.</span><span class="nx">objCount</span><span class="p">}</span><span class="s2">`</span><span class="p">,</span>
      <span class="na">pos</span><span class="p">:</span><span class="nx">s</span><span class="p">.</span><span class="nx">ship</span><span class="p">.</span><span class="nx">pos</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="nx">d</span><span class="p">.</span><span class="nx">scale</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">ship</span><span class="p">.</span><span class="nx">radius</span><span class="p">)),</span>
      <span class="na">vel</span><span class="p">:</span><span class="nx">s</span><span class="p">.</span><span class="nx">ship</span><span class="p">.</span><span class="nx">vel</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="nx">d</span><span class="p">.</span><span class="nx">scale</span><span class="p">(</span><span class="o">-</span><span class="mi">2</span><span class="p">)),</span>
      <span class="na">createTime</span><span class="p">:</span><span class="nx">s</span><span class="p">.</span><span class="nx">time</span><span class="p">,</span>
      <span class="na">thrust</span><span class="p">:</span><span class="kc">false</span><span class="p">,</span>
      <span class="na">angle</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">rotation</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">torque</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">radius</span><span class="p">:</span><span class="mi">3</span>
    <span class="p">}</span>
  <span class="p">}</span>
  <span class="kd">function</span> <span class="nx">createShip</span><span class="p">():</span><span class="nx">Body</span> <span class="p">{</span>
    <span class="k">return</span> <span class="p">{</span>
      <span class="na">id</span><span class="p">:</span> <span class="dl">'</span><span class="s1">ship</span><span class="dl">'</span><span class="p">,</span>
      <span class="na">pos</span><span class="p">:</span> <span class="k">new</span> <span class="nx">Vec</span><span class="p">(</span><span class="nx">CanvasSize</span><span class="o">/</span><span class="mi">2</span><span class="p">,</span><span class="nx">CanvasSize</span><span class="o">/</span><span class="mi">2</span><span class="p">),</span>
      <span class="na">vel</span><span class="p">:</span> <span class="nx">Vec</span><span class="p">.</span><span class="nx">Zero</span><span class="p">,</span>
      <span class="na">thrust</span><span class="p">:</span><span class="kc">false</span><span class="p">,</span>
      <span class="na">angle</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">rotation</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">torque</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">radius</span><span class="p">:</span><span class="mi">20</span><span class="p">,</span>
      <span class="na">createTime</span><span class="p">:</span><span class="mi">0</span>
    <span class="p">}</span>
  <span class="p">}</span>
  <span class="kd">const</span> <span class="nx">initialState</span><span class="p">:</span><span class="nx">State</span> <span class="o">=</span> <span class="p">{</span>
    <span class="na">time</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
    <span class="na">ship</span><span class="p">:</span> <span class="nx">createShip</span><span class="p">(),</span>
    <span class="na">bullets</span><span class="p">:</span> <span class="p">[],</span>
    <span class="na">exit</span><span class="p">:</span> <span class="p">[],</span>
    <span class="na">objCount</span><span class="p">:</span> <span class="mi">0</span>
  <span class="p">}</span>
</code></pre></div></div>
<h1 id="shoot-action">Shoot Action</h1>
<p>We’ll add a new action type and observable for shooting with the space bar:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">class</span> <span class="nx">Shoot</span> <span class="p">{</span> <span class="kd">constructor</span><span class="p">()</span> <span class="p">{}</span> <span class="p">}</span>
  <span class="kd">const</span> <span class="nx">shoot</span> <span class="o">=</span> <span class="nx">keyObservable</span><span class="p">(</span><span class="dl">'</span><span class="s1">keydown</span><span class="dl">'</span><span class="p">,</span><span class="dl">'</span><span class="s1">Space</span><span class="dl">'</span><span class="p">,</span> <span class="p">()</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Shoot</span><span class="p">())</span>
</code></pre></div></div>
<p>And now a function to move objects, same logic as before but now applicable to any <code class="language-plaintext highlighter-rouge">Body</code>:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> <span class="nx">moveObj</span> <span class="o">=</span> <span class="p">(</span><span class="nx">o</span><span class="p">:</span><span class="nx">Body</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="o">&lt;</span><span class="nx">Body</span><span class="o">&gt;</span><span class="p">{</span>
    <span class="p">...</span><span class="nx">o</span><span class="p">,</span>
    <span class="na">rotation</span><span class="p">:</span> <span class="nx">o</span><span class="p">.</span><span class="nx">rotation</span> <span class="o">+</span> <span class="nx">o</span><span class="p">.</span><span class="nx">torque</span><span class="p">,</span>
    <span class="na">angle</span><span class="p">:</span><span class="nx">o</span><span class="p">.</span><span class="nx">angle</span><span class="o">+</span><span class="nx">o</span><span class="p">.</span><span class="nx">rotation</span><span class="p">,</span>
    <span class="na">pos</span><span class="p">:</span><span class="nx">torusWrap</span><span class="p">(</span><span class="nx">o</span><span class="p">.</span><span class="nx">pos</span><span class="p">.</span><span class="nx">sub</span><span class="p">(</span><span class="nx">o</span><span class="p">.</span><span class="nx">vel</span><span class="p">)),</span>
    <span class="na">vel</span><span class="p">:</span><span class="nx">o</span><span class="p">.</span><span class="nx">thrust</span><span class="p">?</span><span class="nx">o</span><span class="p">.</span><span class="nx">vel</span><span class="p">.</span><span class="nx">sub</span><span class="p">(</span><span class="nx">Vec</span><span class="p">.</span><span class="nx">unitVecInDirection</span><span class="p">(</span><span class="nx">o</span><span class="p">.</span><span class="nx">angle</span><span class="p">).</span><span class="nx">scale</span><span class="p">(</span><span class="mf">0.05</span><span class="p">)):</span><span class="nx">o</span><span class="p">.</span><span class="nx">vel</span>
  <span class="p">}</span>
</code></pre></div></div>
<p>And our tick action is a little more complicated now, complicated enough to warrant its own function:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> <span class="nx">tick</span> <span class="o">=</span> <span class="p">(</span><span class="nx">s</span><span class="p">:</span><span class="nx">State</span><span class="p">,</span><span class="nx">elapsed</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
    <span class="kd">const</span> <span class="nx">not</span> <span class="o">=</span> <span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span><span class="na">f</span><span class="p">:(</span><span class="na">x</span><span class="p">:</span><span class="nx">T</span><span class="p">)</span><span class="o">=&gt;</span><span class="nx">boolean</span><span class="p">)</span><span class="o">=&gt;</span><span class="p">(</span><span class="na">x</span><span class="p">:</span><span class="nx">T</span><span class="p">)</span><span class="o">=&gt;!</span><span class="nx">f</span><span class="p">(</span><span class="nx">x</span><span class="p">),</span>
      <span class="nx">expired</span> <span class="o">=</span> <span class="p">(</span><span class="na">b</span><span class="p">:</span><span class="nx">Body</span><span class="p">)</span><span class="o">=&gt;</span><span class="p">(</span><span class="nx">elapsed</span> <span class="o">-</span> <span class="nx">b</span><span class="p">.</span><span class="nx">createTime</span><span class="p">)</span> <span class="o">&gt;</span> <span class="mi">100</span><span class="p">,</span>
      <span class="na">expiredBullets</span><span class="p">:</span><span class="nx">Body</span><span class="p">[]</span> <span class="o">=</span> <span class="nx">s</span><span class="p">.</span><span class="nx">bullets</span><span class="p">.</span><span class="nx">filter</span><span class="p">(</span><span class="nx">expired</span><span class="p">),</span>
      <span class="nx">activeBullets</span> <span class="o">=</span> <span class="nx">s</span><span class="p">.</span><span class="nx">bullets</span><span class="p">.</span><span class="nx">filter</span><span class="p">(</span><span class="nx">not</span><span class="p">(</span><span class="nx">expired</span><span class="p">));</span>
    <span class="k">return</span> <span class="o">&lt;</span><span class="nx">State</span><span class="o">&gt;</span><span class="p">{...</span><span class="nx">s</span><span class="p">,</span> 
      <span class="na">ship</span><span class="p">:</span><span class="nx">moveObj</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">ship</span><span class="p">),</span> 
      <span class="na">bullets</span><span class="p">:</span><span class="nx">activeBullets</span><span class="p">.</span><span class="nx">map</span><span class="p">(</span><span class="nx">moveObj</span><span class="p">),</span> 
      <span class="na">exit</span><span class="p">:</span><span class="nx">expiredBullets</span><span class="p">,</span>
      <span class="na">time</span><span class="p">:</span><span class="nx">elapsed</span>
    <span class="p">}</span>
  <span class="p">}</span>
</code></pre></div></div>
<p>Note that bullets have a life time (presumably they are energy balls that fizzle into space after a certain time).  When a bullet expires it is sent to <code class="language-plaintext highlighter-rouge">exit</code>.</p>

<p>Now adding bullets as they are fired to our state reducer:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> <span class="nx">reduceState</span> <span class="o">=</span> <span class="p">(</span><span class="nx">s</span><span class="p">:</span><span class="nx">State</span><span class="p">,</span> <span class="nx">e</span><span class="p">:</span><span class="nx">Rotate</span><span class="o">|</span><span class="nx">Thrust</span><span class="o">|</span><span class="nx">Tick</span><span class="o">|</span><span class="nx">Shoot</span><span class="p">)</span><span class="o">=&gt;</span>
    <span class="nx">e</span> <span class="k">instanceof</span> <span class="nx">Rotate</span> <span class="p">?</span> <span class="p">{...</span><span class="nx">s</span><span class="p">,</span>
      <span class="na">ship</span><span class="p">:</span> <span class="p">{...</span><span class="nx">s</span><span class="p">.</span><span class="nx">ship</span><span class="p">,</span><span class="na">torque</span><span class="p">:</span><span class="nx">e</span><span class="p">.</span><span class="nx">direction</span><span class="p">}</span>
    <span class="p">}</span> <span class="p">:</span>
    <span class="nx">e</span> <span class="k">instanceof</span> <span class="nx">Thrust</span> <span class="p">?</span> <span class="p">{...</span><span class="nx">s</span><span class="p">,</span>
      <span class="na">ship</span><span class="p">:</span> <span class="p">{...</span><span class="nx">s</span><span class="p">.</span><span class="nx">ship</span><span class="p">,</span> <span class="na">thrust</span><span class="p">:</span><span class="nx">e</span><span class="p">.</span><span class="nx">on</span><span class="p">}</span>
    <span class="p">}</span> <span class="p">:</span>
    <span class="nx">e</span> <span class="k">instanceof</span> <span class="nx">Shoot</span> <span class="p">?</span> <span class="p">{...</span><span class="nx">s</span><span class="p">,</span>
      <span class="na">bullets</span><span class="p">:</span> <span class="nx">s</span><span class="p">.</span><span class="nx">bullets</span><span class="p">.</span><span class="nx">concat</span><span class="p">([</span><span class="nx">createBullet</span><span class="p">(</span><span class="nx">s</span><span class="p">)]),</span>
      <span class="na">objCount</span><span class="p">:</span> <span class="nx">s</span><span class="p">.</span><span class="nx">objCount</span> <span class="o">+</span> <span class="mi">1</span>
    <span class="p">}</span> <span class="p">:</span> 
    <span class="nx">tick</span><span class="p">(</span><span class="nx">s</span><span class="p">,</span><span class="nx">e</span><span class="p">.</span><span class="nx">elapsed</span><span class="p">);</span>
</code></pre></div></div>
<p>We merge the Shoot stream in as before:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="nx">interval</span><span class="p">(</span><span class="mi">10</span><span class="p">).</span><span class="nx">pipe</span><span class="p">(</span>
<span class="p">...</span>
    <span class="nx">merge</span><span class="p">(</span><span class="nx">shoot</span><span class="p">),</span>
<span class="p">...</span>
</code></pre></div></div>
<p>And we tack a bit on to <code class="language-plaintext highlighter-rouge">updateView</code> to draw and remove bullets:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">function</span> <span class="nx">updateView</span><span class="p">(</span><span class="nx">s</span><span class="p">:</span> <span class="nx">State</span><span class="p">)</span> <span class="p">{</span>
<span class="p">...</span>
    <span class="kd">const</span> <span class="nx">svg</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="dl">"</span><span class="s2">svgCanvas</span><span class="dl">"</span><span class="p">)</span><span class="o">!</span><span class="p">;</span>
    <span class="nx">s</span><span class="p">.</span><span class="nx">bullets</span><span class="p">.</span><span class="nx">forEach</span><span class="p">(</span><span class="nx">b</span><span class="o">=&gt;</span><span class="p">{</span>
      <span class="kd">const</span> <span class="nx">createBulletView</span> <span class="o">=</span> <span class="p">()</span><span class="o">=&gt;</span><span class="p">{</span>
        <span class="kd">const</span> <span class="nx">v</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">createElementNS</span><span class="p">(</span><span class="nx">svg</span><span class="p">.</span><span class="nx">namespaceURI</span><span class="p">,</span> <span class="dl">"</span><span class="s2">ellipse</span><span class="dl">"</span><span class="p">)</span><span class="o">!</span><span class="p">;</span>
        <span class="nx">v</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">id</span><span class="dl">"</span><span class="p">,</span><span class="nx">b</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>
        <span class="nx">v</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="dl">"</span><span class="s2">bullet</span><span class="dl">"</span><span class="p">)</span>
        <span class="nx">svg</span><span class="p">.</span><span class="nx">appendChild</span><span class="p">(</span><span class="nx">v</span><span class="p">)</span>
        <span class="k">return</span> <span class="nx">v</span><span class="p">;</span>
      <span class="p">}</span>
      <span class="kd">const</span> <span class="nx">v</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="nx">b</span><span class="p">.</span><span class="nx">id</span><span class="p">)</span> <span class="o">||</span> <span class="nx">createBulletView</span><span class="p">();</span>
      <span class="nx">v</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">cx</span><span class="dl">"</span><span class="p">,</span><span class="nb">String</span><span class="p">(</span><span class="nx">b</span><span class="p">.</span><span class="nx">pos</span><span class="p">.</span><span class="nx">x</span><span class="p">))</span>
      <span class="nx">v</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">cy</span><span class="dl">"</span><span class="p">,</span><span class="nb">String</span><span class="p">(</span><span class="nx">b</span><span class="p">.</span><span class="nx">pos</span><span class="p">.</span><span class="nx">y</span><span class="p">))</span>
      <span class="nx">v</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">rx</span><span class="dl">"</span><span class="p">,</span> <span class="nb">String</span><span class="p">(</span><span class="nx">b</span><span class="p">.</span><span class="nx">radius</span><span class="p">));</span>
      <span class="nx">v</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="dl">"</span><span class="s2">ry</span><span class="dl">"</span><span class="p">,</span> <span class="nb">String</span><span class="p">(</span><span class="nx">b</span><span class="p">.</span><span class="nx">radius</span><span class="p">));</span>
    <span class="p">})</span>
    <span class="nx">s</span><span class="p">.</span><span class="nx">exit</span><span class="p">.</span><span class="nx">forEach</span><span class="p">(</span><span class="nx">o</span><span class="o">=&gt;</span><span class="p">{</span>
      <span class="kd">const</span> <span class="nx">v</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="nx">o</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>
      <span class="k">if</span><span class="p">(</span><span class="nx">v</span><span class="p">)</span> <span class="nx">svg</span><span class="p">.</span><span class="nx">removeChild</span><span class="p">(</span><span class="nx">v</span><span class="p">)</span>
    <span class="p">})</span>
  <span class="p">}</span>
</code></pre></div></div>
<p>Finally, we add a make a quick addition to the CSS so that the bullets are a different colour to the background:</p>
<div class="language-css highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nc">.bullet</span> <span class="p">{</span>
  <span class="py">fill</span><span class="p">:</span> <span class="no">red</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div></div>
<h2 id="collisions">Collisions</h2>
<p>So far the game we have built allows you to hoon around in a space-ship blasting the void with fireballs which is kind of fun, but not very challenging.  The Asteroids game doesn’t really become “Asteroids” until you actually have… asteroids.  Also, you should be able to break them up with your blaster and crashing into them should end the game.  Here’s a preview:</p>

<p><a href="https://stackblitz.com/edit/asteroids05?file=index.ts"><img src="AsteroidsComplete.gif" alt="Spaceship flying" /></a></p>

<p>Before we go forward, let’s put all the magic numbers that are starting to permeate our code in one, immutable place:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> 
    <span class="nx">Constants</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">CanvasSize</span><span class="p">:</span> <span class="mi">600</span><span class="p">,</span>
      <span class="na">BulletExpirationTime</span><span class="p">:</span> <span class="mi">1000</span><span class="p">,</span>
      <span class="na">BulletRadius</span><span class="p">:</span> <span class="mi">3</span><span class="p">,</span>
      <span class="na">BulletVelocity</span><span class="p">:</span> <span class="mi">2</span><span class="p">,</span>
      <span class="na">StartRockRadius</span><span class="p">:</span> <span class="mi">30</span><span class="p">,</span>
      <span class="na">StartRocksCount</span><span class="p">:</span> <span class="mi">5</span><span class="p">,</span>
      <span class="na">RotationAcc</span><span class="p">:</span> <span class="mf">0.1</span><span class="p">,</span>
      <span class="na">ThrustAcc</span><span class="p">:</span> <span class="mf">0.1</span><span class="p">,</span>
      <span class="na">StartTime</span><span class="p">:</span> <span class="mi">0</span>
    <span class="p">}</span> <span class="k">as</span> <span class="kd">const</span>
</code></pre></div></div>

<h1 id="initial-state">Initial State</h1>
<p>We will need to store two new pieces of state: the collection of asteroids (<code class="language-plaintext highlighter-rouge">rocks</code>) which is another array of <code class="language-plaintext highlighter-rouge">Body</code>, just like bullets; and also a boolean that will become <code class="language-plaintext highlighter-rouge">true</code> when the game ends due to collision between the ship and a rock.</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">type</span> <span class="nx">State</span> <span class="o">=</span> <span class="nb">Readonly</span><span class="o">&lt;</span><span class="p">{</span>
    <span class="p">...</span>
    <span class="na">rocks</span><span class="p">:</span><span class="nx">ReadonlyArray</span><span class="o">&lt;</span><span class="nx">Body</span><span class="o">&gt;</span><span class="p">,</span>
    <span class="na">gameOver</span><span class="p">:</span><span class="nx">boolean</span>
  <span class="p">}</span><span class="o">&gt;</span>
</code></pre></div></div>
<p>Since bullets and rocks are both just circular <code class="language-plaintext highlighter-rouge">Body</code>s with constant velocity, we can generalise what was previously the <code class="language-plaintext highlighter-rouge">createBullet</code> function to create either:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">type</span> <span class="nx">ViewType</span> <span class="o">=</span> <span class="dl">'</span><span class="s1">ship</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">rock</span><span class="dl">'</span> <span class="o">|</span> <span class="dl">'</span><span class="s1">bullet</span><span class="dl">'</span>
  <span class="kd">const</span> <span class="nx">createCircle</span> <span class="o">=</span> <span class="p">(</span><span class="nx">viewType</span><span class="p">:</span> <span class="nx">ViewType</span><span class="p">)</span><span class="o">=&gt;</span> <span class="p">(</span><span class="nx">oid</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span><span class="o">=&gt;</span> <span class="p">(</span><span class="nx">time</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span><span class="o">=&gt;</span> <span class="p">(</span><span class="nx">radius</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span><span class="o">=&gt;</span> <span class="p">(</span><span class="nx">pos</span><span class="p">:</span><span class="nx">Vec</span><span class="p">)</span><span class="o">=&gt;</span> <span class="p">(</span><span class="nx">vel</span><span class="p">:</span><span class="nx">Vec</span><span class="p">)</span><span class="o">=&gt;</span>
    <span class="o">&lt;</span><span class="nx">Body</span><span class="o">&gt;</span><span class="p">{</span>
      <span class="na">createTime</span><span class="p">:</span> <span class="nx">time</span><span class="p">,</span>
      <span class="na">pos</span><span class="p">:</span><span class="nx">pos</span><span class="p">,</span>
      <span class="na">vel</span><span class="p">:</span><span class="nx">vel</span><span class="p">,</span>
      <span class="na">thrust</span><span class="p">:</span> <span class="kc">false</span><span class="p">,</span>
      <span class="na">angle</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span> <span class="na">rotation</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span> <span class="na">torque</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">radius</span><span class="p">:</span> <span class="nx">radius</span><span class="p">,</span>
      <span class="na">id</span><span class="p">:</span> <span class="nx">viewType</span><span class="o">+</span><span class="nx">oid</span><span class="p">,</span>
      <span class="na">viewType</span><span class="p">:</span> <span class="nx">viewType</span>
    <span class="p">};</span>
</code></pre></div></div>

<p>Our initial state is going to include several rocks drifting in random directions, as follows:</p>
<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span>
    <span class="nx">startRocks</span> <span class="o">=</span> <span class="p">[...</span><span class="nb">Array</span><span class="p">(</span><span class="nx">Constants</span><span class="p">.</span><span class="nx">StartRocksCount</span><span class="p">)]</span>
      <span class="p">.</span><span class="nx">map</span><span class="p">((</span><span class="nx">_</span><span class="p">,</span><span class="nx">i</span><span class="p">)</span><span class="o">=&gt;</span><span class="nx">createCircle</span><span class="p">(</span><span class="dl">"</span><span class="s2">rock</span><span class="dl">"</span><span class="p">)(</span><span class="nx">i</span><span class="p">)</span>
         <span class="p">(</span><span class="nx">Constants</span><span class="p">.</span><span class="nx">StartTime</span><span class="p">)(</span><span class="nx">Constants</span><span class="p">.</span><span class="nx">StartRockRadius</span><span class="p">)(</span><span class="nx">Vec</span><span class="p">.</span><span class="nx">Zero</span><span class="p">)</span>
         <span class="p">(</span><span class="k">new</span> <span class="nx">Vec</span><span class="p">(</span><span class="mf">0.5</span> <span class="o">-</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">random</span><span class="p">(),</span> <span class="mf">0.5</span> <span class="o">-</span> <span class="nb">Math</span><span class="p">.</span><span class="nx">random</span><span class="p">()))),</span>
    <span class="nx">initialState</span><span class="p">:</span><span class="nx">State</span> <span class="o">=</span> <span class="p">{</span>
      <span class="na">time</span><span class="p">:</span><span class="mi">0</span><span class="p">,</span>
      <span class="na">ship</span><span class="p">:</span> <span class="nx">createShip</span><span class="p">(),</span>
      <span class="na">bullets</span><span class="p">:</span> <span class="p">[],</span>
      <span class="na">rocks</span><span class="p">:</span> <span class="nx">startRocks</span><span class="p">,</span>
      <span class="na">exit</span><span class="p">:</span> <span class="p">[],</span>
      <span class="na">objCount</span><span class="p">:</span> <span class="nx">Constants</span><span class="p">.</span><span class="nx">StartRocksCount</span><span class="p">,</span>
      <span class="na">gameOver</span><span class="p">:</span> <span class="kc">false</span>
    <span class="p">}</span>
</code></pre></div></div>

<h1 id="reducing-state-1">Reducing State</h1>

<p>Our <code class="language-plaintext highlighter-rouge">tick</code> function is more or less the same as above, but it will apply one more transformation to the state that it returns, by applying the following function.  This function checks for collisions between the ship and rocks, and also between bullets and rocks.</p>

<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>    <span class="c1">// check a State for collisions:</span>
    <span class="c1">//   bullets destroy rocks spawning smaller ones</span>
    <span class="c1">//   ship colliding with rock ends game</span>
    <span class="kd">const</span> <span class="nx">handleCollisions</span> <span class="o">=</span> <span class="p">(</span><span class="nx">s</span><span class="p">:</span><span class="nx">State</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">{</span>
      <span class="kd">const</span>
        <span class="c1">// Some array utility functions</span>
        <span class="nx">not</span> <span class="o">=</span> <span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">(</span><span class="na">f</span><span class="p">:(</span><span class="na">x</span><span class="p">:</span><span class="nx">T</span><span class="p">)</span><span class="o">=&gt;</span><span class="nx">boolean</span><span class="p">)</span><span class="o">=&gt;</span><span class="p">(</span><span class="na">x</span><span class="p">:</span><span class="nx">T</span><span class="p">)</span><span class="o">=&gt;!</span><span class="nx">f</span><span class="p">(</span><span class="nx">x</span><span class="p">),</span>
        <span class="nx">mergeMap</span> <span class="o">=</span> <span class="o">&lt;</span><span class="nx">T</span><span class="p">,</span> <span class="nx">U</span><span class="o">&gt;</span><span class="p">(</span>
          <span class="na">a</span><span class="p">:</span> <span class="nx">ReadonlyArray</span><span class="o">&lt;</span><span class="nx">T</span><span class="o">&gt;</span><span class="p">,</span>
          <span class="na">f</span><span class="p">:</span> <span class="p">(</span><span class="na">a</span><span class="p">:</span> <span class="nx">T</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">ReadonlyArray</span><span class="o">&lt;</span><span class="nx">U</span><span class="o">&gt;</span>
        <span class="p">)</span> <span class="o">=&gt;</span> <span class="nb">Array</span><span class="p">.</span><span class="nx">prototype</span><span class="p">.</span><span class="nx">concat</span><span class="p">(...</span><span class="nx">a</span><span class="p">.</span><span class="nx">map</span><span class="p">(</span><span class="nx">f</span><span class="p">)),</span>

        <span class="nx">bodiesCollided</span> <span class="o">=</span> <span class="p">([</span><span class="nx">a</span><span class="p">,</span><span class="nx">b</span><span class="p">]:[</span><span class="nx">Body</span><span class="p">,</span><span class="nx">Body</span><span class="p">])</span> <span class="o">=&gt;</span> <span class="nx">a</span><span class="p">.</span><span class="nx">pos</span><span class="p">.</span><span class="nx">sub</span><span class="p">(</span><span class="nx">b</span><span class="p">.</span><span class="nx">pos</span><span class="p">).</span><span class="nx">len</span><span class="p">()</span> <span class="o">&lt;</span> <span class="nx">a</span><span class="p">.</span><span class="nx">radius</span> <span class="o">+</span> <span class="nx">b</span><span class="p">.</span><span class="nx">radius</span><span class="p">,</span>
        <span class="nx">shipCollided</span> <span class="o">=</span> <span class="nx">s</span><span class="p">.</span><span class="nx">rocks</span><span class="p">.</span><span class="nx">filter</span><span class="p">(</span><span class="nx">r</span><span class="o">=&gt;</span><span class="nx">bodiesCollided</span><span class="p">([</span><span class="nx">s</span><span class="p">.</span><span class="nx">ship</span><span class="p">,</span><span class="nx">r</span><span class="p">])).</span><span class="nx">length</span> <span class="o">&gt;</span> <span class="mi">0</span><span class="p">,</span>
        <span class="nx">allBulletsAndRocks</span> <span class="o">=</span> <span class="nx">mergeMap</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">bullets</span><span class="p">,</span> <span class="nx">b</span><span class="o">=&gt;</span> <span class="nx">s</span><span class="p">.</span><span class="nx">rocks</span><span class="p">.</span><span class="nx">map</span><span class="p">(</span><span class="nx">r</span><span class="o">=&gt;</span><span class="p">([</span><span class="nx">b</span><span class="p">,</span><span class="nx">r</span><span class="p">]))),</span>
        <span class="nx">collidedBulletsAndRocks</span> <span class="o">=</span> <span class="nx">allBulletsAndRocks</span><span class="p">.</span><span class="nx">filter</span><span class="p">(</span><span class="nx">bodiesCollided</span><span class="p">),</span>
        <span class="nx">collidedBullets</span> <span class="o">=</span> <span class="nx">collidedBulletsAndRocks</span><span class="p">.</span><span class="nx">map</span><span class="p">(([</span><span class="nx">bullet</span><span class="p">,</span><span class="nx">_</span><span class="p">])</span><span class="o">=&gt;</span><span class="nx">bullet</span><span class="p">),</span>
        <span class="nx">collidedRocks</span> <span class="o">=</span> <span class="nx">collidedBulletsAndRocks</span><span class="p">.</span><span class="nx">map</span><span class="p">(([</span><span class="nx">_</span><span class="p">,</span><span class="nx">rock</span><span class="p">])</span><span class="o">=&gt;</span><span class="nx">rock</span><span class="p">),</span>

        <span class="c1">// spawn two children for each collided rock above a certain size</span>
        <span class="nx">child</span> <span class="o">=</span> <span class="p">(</span><span class="na">r</span><span class="p">:</span><span class="nx">Body</span><span class="p">,</span><span class="na">dir</span><span class="p">:</span><span class="kr">number</span><span class="p">)</span><span class="o">=&gt;</span><span class="p">({</span>
          <span class="na">radius</span><span class="p">:</span> <span class="nx">r</span><span class="p">.</span><span class="nx">radius</span><span class="o">/</span><span class="mi">2</span><span class="p">,</span>
          <span class="na">pos</span><span class="p">:</span><span class="nx">r</span><span class="p">.</span><span class="nx">pos</span><span class="p">,</span>
          <span class="na">vel</span><span class="p">:</span><span class="nx">r</span><span class="p">.</span><span class="nx">vel</span><span class="p">.</span><span class="nx">ortho</span><span class="p">().</span><span class="nx">scale</span><span class="p">(</span><span class="nx">dir</span><span class="p">)</span>
        <span class="p">}),</span>
        <span class="nx">spawnChildren</span> <span class="o">=</span> <span class="p">(</span><span class="na">r</span><span class="p">:</span><span class="nx">Body</span><span class="p">)</span><span class="o">=&gt;</span>
                              <span class="nx">r</span><span class="p">.</span><span class="nx">radius</span> <span class="o">&gt;=</span> <span class="nx">Constants</span><span class="p">.</span><span class="nx">StartRockRadius</span><span class="o">/</span><span class="mi">4</span> 
                              <span class="p">?</span> <span class="p">[</span><span class="nx">child</span><span class="p">(</span><span class="nx">r</span><span class="p">,</span><span class="mi">1</span><span class="p">),</span> <span class="nx">child</span><span class="p">(</span><span class="nx">r</span><span class="p">,</span><span class="o">-</span><span class="mi">1</span><span class="p">)]</span> <span class="p">:</span> <span class="p">[],</span>
        <span class="nx">newRocks</span> <span class="o">=</span> <span class="nx">mergeMap</span><span class="p">(</span><span class="nx">collidedRocks</span><span class="p">,</span> <span class="nx">spawnChildren</span><span class="p">)</span>
          <span class="p">.</span><span class="nx">map</span><span class="p">((</span><span class="nx">r</span><span class="p">,</span><span class="nx">i</span><span class="p">)</span><span class="o">=&gt;</span><span class="nx">createCircle</span><span class="p">(</span><span class="dl">'</span><span class="s1">rock</span><span class="dl">'</span><span class="p">)(</span><span class="nx">s</span><span class="p">.</span><span class="nx">objCount</span> <span class="o">+</span> <span class="nx">i</span><span class="p">)(</span><span class="nx">s</span><span class="p">.</span><span class="nx">time</span><span class="p">)(</span><span class="nx">r</span><span class="p">.</span><span class="nx">radius</span><span class="p">)(</span><span class="nx">r</span><span class="p">.</span><span class="nx">pos</span><span class="p">)(</span><span class="nx">r</span><span class="p">.</span><span class="nx">vel</span><span class="p">)),</span>

        <span class="c1">// search for a body by id in an array</span>
        <span class="nx">elem</span> <span class="o">=</span> <span class="p">(</span><span class="na">a</span><span class="p">:</span><span class="nx">ReadonlyArray</span><span class="o">&lt;</span><span class="nx">Body</span><span class="o">&gt;</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="na">e</span><span class="p">:</span><span class="nx">Body</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="nx">a</span><span class="p">.</span><span class="nx">findIndex</span><span class="p">(</span><span class="nx">b</span><span class="o">=&gt;</span><span class="nx">b</span><span class="p">.</span><span class="nx">id</span> <span class="o">===</span> <span class="nx">e</span><span class="p">.</span><span class="nx">id</span><span class="p">)</span> <span class="o">&gt;=</span> <span class="mi">0</span><span class="p">,</span>
        <span class="c1">// array a except anything in b</span>
        <span class="nx">except</span> <span class="o">=</span> <span class="p">(</span><span class="na">a</span><span class="p">:</span><span class="nx">ReadonlyArray</span><span class="o">&lt;</span><span class="nx">Body</span><span class="o">&gt;</span><span class="p">)</span> <span class="o">=&gt;</span> <span class="p">(</span><span class="na">b</span><span class="p">:</span><span class="nx">Body</span><span class="p">[])</span> <span class="o">=&gt;</span> <span class="nx">a</span><span class="p">.</span><span class="nx">filter</span><span class="p">(</span><span class="nx">not</span><span class="p">(</span><span class="nx">elem</span><span class="p">(</span><span class="nx">b</span><span class="p">)))</span>
      
      <span class="k">return</span> <span class="o">&lt;</span><span class="nx">State</span><span class="o">&gt;</span><span class="p">{</span>
        <span class="p">...</span><span class="nx">s</span><span class="p">,</span>
        <span class="na">bullets</span><span class="p">:</span> <span class="nx">except</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">bullets</span><span class="p">)(</span><span class="nx">collidedBullets</span><span class="p">),</span>
        <span class="na">rocks</span><span class="p">:</span> <span class="nx">except</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">rocks</span><span class="p">)(</span><span class="nx">collidedRocks</span><span class="p">).</span><span class="nx">concat</span><span class="p">(</span><span class="nx">newRocks</span><span class="p">),</span>
        <span class="na">exit</span><span class="p">:</span> <span class="nx">s</span><span class="p">.</span><span class="nx">exit</span><span class="p">.</span><span class="nx">concat</span><span class="p">(</span><span class="nx">collidedBullets</span><span class="p">,</span><span class="nx">collidedRocks</span><span class="p">),</span>
        <span class="na">objCount</span><span class="p">:</span> <span class="nx">s</span><span class="p">.</span><span class="nx">objCount</span> <span class="o">+</span> <span class="nx">newRocks</span><span class="p">.</span><span class="nx">length</span><span class="p">,</span>
        <span class="na">gameOver</span><span class="p">:</span> <span class="nx">shipCollided</span>
      <span class="p">}</span>
    <span class="p">},</span>
</code></pre></div></div>

<h1 id="final-view">Final View</h1>

<p>Finally, we need to update <code class="language-plaintext highlighter-rouge">updateView</code> function.  Again, the view update is the one place in our program where we allow imperative style, effectful code.  Called only from the subscribe at the very end of our Observable chain, and not mutating any state that is read anywhere else in the Observable, we ensure that is not the source of any but the simplest display bugs, which we can hopefully diagnose with local inspection of this one function.</p>

<p>First, we need to update the visuals for each of the rocks, but these are the same as bullets.  The second, slightly bigger, change, is simply to display the text “Game Over” on <code class="language-plaintext highlighter-rouge">s.gameover</code> true.</p>

<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">function</span> <span class="nx">updateView</span><span class="p">(</span><span class="nx">s</span><span class="p">:</span> <span class="nx">State</span><span class="p">)</span> <span class="p">{</span>
  <span class="p">...</span>
    <span class="nx">s</span><span class="p">.</span><span class="nx">bullets</span><span class="p">.</span><span class="nx">forEach</span><span class="p">(</span><span class="nx">updateBodyView</span><span class="p">);</span>
    <span class="nx">s</span><span class="p">.</span><span class="nx">rocks</span><span class="p">.</span><span class="nx">forEach</span><span class="p">(</span><span class="nx">updateBodyView</span><span class="p">);</span>
    <span class="nx">s</span><span class="p">.</span><span class="nx">exit</span><span class="p">.</span><span class="nx">forEach</span><span class="p">(</span><span class="nx">o</span><span class="o">=&gt;</span><span class="p">{</span>
      <span class="kd">const</span> <span class="nx">v</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="nx">o</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span>
      <span class="k">if</span><span class="p">(</span><span class="nx">v</span><span class="p">)</span> <span class="nx">svg</span><span class="p">.</span><span class="nx">removeChild</span><span class="p">(</span><span class="nx">v</span><span class="p">);</span>
    <span class="p">})</span>
    <span class="k">if</span><span class="p">(</span><span class="nx">s</span><span class="p">.</span><span class="nx">gameOver</span><span class="p">)</span> <span class="p">{</span>
      <span class="nx">subscription</span><span class="p">.</span><span class="nx">unsubscribe</span><span class="p">();</span>
      <span class="kd">const</span> <span class="nx">v</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">createElementNS</span><span class="p">(</span><span class="nx">svg</span><span class="p">.</span><span class="nx">namespaceURI</span><span class="p">,</span> <span class="dl">"</span><span class="s2">text</span><span class="dl">"</span><span class="p">)</span><span class="o">!</span><span class="p">;</span>
      <span class="nx">attr</span><span class="p">(</span><span class="nx">v</span><span class="p">,{</span>
        <span class="na">x</span><span class="p">:</span> <span class="nx">Constants</span><span class="p">.</span><span class="nx">CanvasSize</span><span class="o">/</span><span class="mi">6</span><span class="p">,</span>
        <span class="na">y</span><span class="p">:</span> <span class="nx">Constants</span><span class="p">.</span><span class="nx">CanvasSize</span><span class="o">/</span><span class="mi">2</span><span class="p">,</span>
        <span class="na">class</span><span class="p">:</span> <span class="dl">"</span><span class="s2">gameover</span><span class="dl">"</span>
      <span class="p">});</span>
      <span class="nx">v</span><span class="p">.</span><span class="nx">textContent</span> <span class="o">=</span> <span class="dl">"</span><span class="s2">Game Over</span><span class="dl">"</span><span class="p">;</span>
      <span class="nx">svg</span><span class="p">.</span><span class="nx">appendChild</span><span class="p">(</span><span class="nx">v</span><span class="p">);</span>
    <span class="p">}</span>
  <span class="p">}</span>
</code></pre></div></div>

<p>where we’ve created a little helper function <code class="language-plaintext highlighter-rouge">attr</code> to bulk set properties on an <code class="language-plaintext highlighter-rouge">Element</code>:</p>

<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span>
    <span class="nx">attr</span> <span class="o">=</span> <span class="p">(</span><span class="nx">e</span><span class="p">:</span><span class="nx">Element</span><span class="p">,</span> <span class="nx">o</span><span class="p">:</span><span class="nb">Object</span><span class="p">)</span> <span class="o">=&gt;</span>
      <span class="p">{</span> <span class="k">for</span><span class="p">(</span><span class="kd">const</span> <span class="nx">k</span> <span class="k">in</span> <span class="nx">o</span><span class="p">)</span> <span class="nx">e</span><span class="p">.</span><span class="nx">setAttribute</span><span class="p">(</span><span class="nx">k</span><span class="p">,</span><span class="nb">String</span><span class="p">(</span><span class="nx">o</span><span class="p">[</span><span class="nx">k</span><span class="p">]))</span> <span class="p">},</span>
</code></pre></div></div>

<p>The other thing happening at game over, is the call to <code class="language-plaintext highlighter-rouge">subscription.unsubscribe</code>.  This <code class="language-plaintext highlighter-rouge">subscription</code> is the object returned by the subscribe call on our main Observable:</p>

<div class="language-typescript highlighter-rouge"><div class="highlight"><pre class="highlight"><code>  <span class="kd">const</span> <span class="nx">subscription</span> <span class="o">=</span> <span class="nx">interval</span><span class="p">(</span><span class="mi">10</span><span class="p">).</span><span class="nx">pipe</span><span class="p">(</span>
    <span class="nx">map</span><span class="p">(</span><span class="nx">elapsed</span><span class="o">=&gt;</span><span class="k">new</span> <span class="nx">Tick</span><span class="p">(</span><span class="nx">elapsed</span><span class="p">)),</span>
    <span class="nx">merge</span><span class="p">(</span>
      <span class="nx">startLeftRotate</span><span class="p">,</span><span class="nx">startRightRotate</span><span class="p">,</span><span class="nx">stopLeftRotate</span><span class="p">,</span><span class="nx">stopRightRotate</span><span class="p">),</span>
    <span class="nx">merge</span><span class="p">(</span><span class="nx">startThrust</span><span class="p">,</span><span class="nx">stopThrust</span><span class="p">),</span>
    <span class="nx">merge</span><span class="p">(</span><span class="nx">shoot</span><span class="p">),</span>
    <span class="nx">scan</span><span class="p">(</span><span class="nx">reduceState</span><span class="p">,</span> <span class="nx">initialState</span><span class="p">)</span>
    <span class="p">).</span><span class="nx">subscribe</span><span class="p">(</span><span class="nx">updateView</span><span class="p">);</span>
</code></pre></div></div>

<p>Finally, we need to make a couple more additions to the CSS to display the rocks and game over text:</p>

<pre><code class="language-CSS">.rock {
  fill: burlywood;
}
.gameover {
  font-family: sans-serif;
  font-size: 80px;
  fill: red;
}
</code></pre>

<p>At this point we have more-or-less all the elements of a game.  The implementation above could be extended quite a lot.  For example, we could add score, ability to restart the game, multiple lives, perhaps some more physics.  But generally, these are just extensions to the framework above: manipulation and then display of additional state.</p>

<p>The key thing is that the observable has allowed us to keep well separated state management (model), its input and manipulation (control) and the visuals (view).  Further extensions are just additions within each of these elements - and doing so should not add greatly to the complexity.</p>

<p>I invite you to click through on the animations above, to the live code editor where you can extend or refine the framework I’ve started.</p>

  </div>

</article>

      </div>
    </main><footer class="site-footer h-card">
    <data class="u-url" href="/"></data>
  
    <div class="wrapper">
  
      <div class="footer-col-wrapper">
        <div class="footer-col">
          <!-- <p class="feed-subscribe">
            <a href="/feed.xml">
              <svg class="svg-icon orange">
                <use xlink:href="/assets/minima-social-icons.svg#rss"></use>
              </svg><span>Subscribe</span>
            </a>
          </p> -->
        </div>
        <div class="footer-col">
          <p>Examples and tutorials for various programming paradigms.</p>
        </div>
      </div>
  
      <div class="social-links"><ul class="social-media-list"><li><a href="https://github.com/tgdwyer"><svg class="svg-icon"><use xlink:href="/assets/minima-social-icons.svg#github"></use></svg> <span class="username">tgdwyer</span></a></li><li><a href="https://www.twitter.com/immersivecola"><svg class="svg-icon"><use xlink:href="/assets/minima-social-icons.svg#twitter"></use></svg> <span class="username">immersivecola</span></a></li></ul>
</div>
  
    </div>
  
  </footer></body>


