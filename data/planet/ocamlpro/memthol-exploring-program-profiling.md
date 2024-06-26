---
title: 'Memthol: exploring program profiling'
description: "Memthol is a visualizer and analyzer for program profiling. It works
  on memory dumps containing information about the size and (de)allocation date of
  part of the allocations performed by some execution of a program. For information
  regarding building memthol, features, browser compatibility\u2026 refer..."
url: https://ocamlpro.com/blog/2020_12_01_memthol_exploring_program_profiling
date: 2020-12-01T13:19:46-00:00
preview_image: URL_de_votre_image
authors:
- "\n    Adrien Champion\n  "
source:
---

<p><img src="https://ocamlpro.com/blog/assets/img/banner_memprof_banniere_blue.png" alt=""/></p>
<p><em>Memthol</em> is a visualizer and analyzer for program profiling. It works on memory <em>dumps</em> containing information about the size and (de)allocation date of part of the allocations performed by some execution of a program.</p>
<blockquote>
<p>For information regarding building memthol, features, browser compatibility&hellip; refer to the <a href="https://github.com/OCamlPro/memthol">memthol github repository</a>. *Please note that Memthol, as a side project, is a work in progress that remains in beta status for now. *</p>
</blockquote>
<p><img src="https://raw.githubusercontent.com/OCamlPro/memthol/master/rsc/example.png" alt=""/></p>
<h4>Memthol's background</h4>
<p>The Memthol work was started more than a year ago (we had published a short introductory paper at the <a href="https://jfla.inria.fr/jfla2020.html">JFLA2020</a>). The whole idea was to use the previous work originally achieved on <a href="https://memprof.typerex.org/">ocp-memprof</a>, and look for some extra funding to achieve a usable and industrial version.Then came the excellent <a href="https://blog.janestreet.com/finding-memory-leaks-with-memtrace/">memtrace profiler</a> by Jane Street's team (congrats!)Memthol is a self-funded side project, that we think it still is worth giving to the OCaml community. Its approach is valuable, and can be complementary. It is released under the free GPL licence v3.</p>
<h4>Memthol's versatility: supporting memtrace's dump format</h4>
<p>The memtrace format is nicely designed and polished enough to be considered a future standard for other tools.This is why Memthol supports Jane Street's <em>dumper</em> format, instead of our own dumper library's.</p>
<h4>Why choose Rust to implement Memthol?</h4>
<p>We've been exploring the Rust language for more than a year now.The Memthol work was the opportunity to further explore this state-of-the-art language. <em>We are open to extra funding, to deepen the Memthol work should industrial users be interested.</em></p>
<h4>Memthol's How-to</h4>
<blockquote>
<p>The following steps are from the <a href="https://ocamlpro.github.io/memthol/mini_tutorial/">Memthol Github howto</a>.</p>
<ul>
<li><strong>1.</strong> <a href="https://ocamlpro.github.io/memthol/mini_tutorial/basics.html">Introduction</a>
</li>
<li><strong>2.</strong> <a href="https://ocamlpro.github.io/memthol/mini_tutorial/charts.html">Basics</a>
</li>
<li><strong>3.</strong> <a href="https://ocamlpro.github.io/memthol/mini_tutorial/global_settings.html">Charts</a>
</li>
<li><strong>4.</strong> <a href="https://ocamlpro.github.io/memthol/mini_tutorial/callstack_filters.html">Global Settings</a>
</li>
<li><strong>5.</strong> <a href="https://ocamlpro.github.io/memthol/mini_tutorial/">Callstack Filters</a>
</li>
</ul>
</blockquote>
<h2>Introduction</h2>
<p>This tutorial deals with the BUI ( <strong>B</strong>rowser <strong>U</strong>ser <strong>I</strong>nterface) aspect of the profiling. How the dumps are generated is outside of the scope of this document. Currently, memthol accepts memory dumps produced by <em>[Memtrace]</em>(https://blog.janestreet.com/finding-memory-leaks-with-memtrace) (github repository <a href="https://github.com/janestreet/memtrace">here</a>). A memtrace dump for a program execution is a single <a href="https://diamon.org/ctf"> <strong>C</strong>ommon <strong>T</strong>race <strong>F</strong>ormat</a> (CTF) file.</p>
<p>This tutorial uses CTF files from the memthol repository. All paths mentioned in the examples are from its root.</p>
<p>Memthol is written in Rust and is composed of</p>
<ul>
<li>a server, written in pure Rust, and
</li>
<li>a client, written in Rust and compiled to web assembly.
</li>
</ul>
<p>The server contains the client, which it will serve at some address on some port when launched.</p>
<h3>Running Memthol</h3>
<p>Memthol must be given a path to a CTF file generated by memtrace.</p>
<pre><code class="language-shell-session">&gt; ls rsc/dumps/ctf/flamba.ctf
rsc/dumps/ctf/flamba.ctf
&gt; memthol rsc/dumps/ctf/flamba.ctf
|===| Starting
| url: http://localhost:7878
| target: `rsc/dumps/ctf/flamba.ctf`
|===|
</code></pre>
<h2>Basics</h2>
<p>Our running example in this section will be <code>rsc/dumps/mini_ae.ctf</code>:</p>
<pre><code class="language-shell-session">&#10095; memthol --filter_gen none rsc/dumps/ctf/mini_ae.ctf
|===| Starting
| url: http://localhost:7878
| target: `rsc/dumps/ctf/mini_ae.ctf`
|===|
</code></pre>
<p>Notice the odd <code>--filter_gen none</code> passed to memthol. Ignore it for now, it will be discussed later in this section.</p>
<p>Once memthol is running, <code>http://localhost:7878/</code> (here) will lead you to memthol's BUI, which should look something like this:</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/default.png" alt=""/></p>
<p>Click on the orange <strong>everything</strong> tab at the bottom left of the screen.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/three_parts.png" alt=""/></p>
<p>Memthol's interface is split in three parts:</p>
<ul>
<li>the central, main part displays charts. There is only one here, showing the evolution of the program's total memory size over time based on the memory dump.
</li>
<li>the header gives statistics about the memory dump and handles general settings. There is currently only one, the <em>time window</em>.- the footer controls your <em>filters</em> (there is only one here), which we are going to discuss right now.
</li>
</ul>
<h3>Filters</h3>
<p><em>Filters</em> allow to split allocations and display them separately. A filter is essentially a set of allocations. Memthol has two built-in filters. The first one is the <strong>everything</strong> filter. You cannot really do anything with it except for changing its name and color using the filter settings in the footer.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/everything_name_color.png" alt=""/></p>
<p>Notice that when a filter is modified, two buttons appear in the top-left part of the footer. The first reverts the changes while the second one saves them. Let's save these changes.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/everything_saved.png" alt=""/></p>
<p>The <strong>everything</strong> filter always contains all allocations in the memory dump. It cannot be changed besides the cosmetic changes we just did. These changes are reverted in the rest of the section.</p>
<h3>Custom Filters</h3>
<p>Let's create a new filter using the <code>+</code> add button in the top-right part of the footer.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/new_filter.png" alt=""/></p>
<p>Notice that, unlike <strong>everything</strong>, the settings for our new filter have a <strong>Catch allocation if &hellip;</strong> (empty) section with a <code>+</code> add button. Let's click on that.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/new_sub_filter.png" alt=""/></p>
<p>This adds a criterion to our filter. Let's modify it so that the our filter catches everything of size greater than zero machine words, rename the filter, and save these changes.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/new_filter_1.png" alt=""/></p>
<p>The tab for our filter now shows <strong>(3)</strong> next to its name, indicating that this filter catches 3 allocations, which is all the allocations of the (tiny) dump.</p>
<p>Now, create a new filter and modify it so that it catches allocations made in file <code>weak.ml</code>. This requires</p>
<ul>
<li>creating a filter,
</li>
<li>adding a criterion to that filter,
</li>
<li>switching it from <code>size</code> to <code>callstack</code>
</li>
<li>removing the trailing <code>**</code> (anything) by erasing it,
</li>
<li>write <code>weak.ml</code> as the last file that should appear in the callstack.&gt;
</li>
</ul>
<p>After saving it, you should get the following.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/new_filter_2.png" alt=""/></p>
<p>Sadly, this filter does not match anything, although some allocations fit this filter. This is because a <strong>custom filter</strong> <code>F</code> &ldquo;catches&quot; an allocation if</p>
<ul>
<li>all of the criteria of <code>F</code> are true for this allocation, and
</li>
<li>the allocation is not caught by any <strong>custom</strong> filter at the left of <code>F</code> (note that the <strong>everything</strong> filter is not a <strong>custom filter</strong>).
</li>
</ul>
<p>In other words, all allocations go through the list of custom filters from left to right, and are caught by the first filter such that all of its criteria are true for this allocation. As such, it is similar to switch/case and pattern matching.</p>
<p>Let's move our new filter to the left by clicking the left arrow next to it, and save the change.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/new_filter_3.png" alt=""/></p>
<p>Nice.</p>
<p>You can remove a filter by selecting it and clicking the <code>-</code> remove button in the top-right part of the footer, next to the <code>+</code> add filter button. This only works for <strong>custom</strong> filters, you cannot remove built-in filters.</p>
<p>Now, remove the first filter we created (size &ge; 0), which should give you this:</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/new_filter_4.png" alt=""/></p>
<p>Out of nowhere, we get the second and last built-in filter: <strong>catch-all</strong>. When some allocations are not caught by any of your filters, they will end up in this filter. <strong>Catch-all</strong> is not visible when it does not catch any allocation, which is why it was (mostly) not visible until now. The filter we wrote previously where catching all the allocations.</p>
<blockquote>
<p>In the switch/case analogy, <strong>catch-all</strong> is the <code>else</code>/<code>default</code> branch. In pattern matching, it would be a trailing wildcard <code>_</code>.</p>
</blockquote>
<p>So, <code>weak.ml</code> only catches one of the three allocations: <strong>catch-all</strong> appears and indicates it matches the remaining two.</p>
<blockquote>
<p>It is also possible to write filter criteria over allocations' callstacks. This is discussed in the <a href="https://ocamlpro.github.io/memthol/mini_tutorial/callstack_filters.html">Callstack Filters Section</a>.</p>
</blockquote>
<h3>Filter Generation</h3>
<p>When we launched this section's running example, we passed <code>--filter_gen none</code> to memthol. This is because, by default, memthol will run <em>automatic filter generation</em> which scans allocations and generates filters. The default (and currently only) one creates one filter per allocation-site file.</p>
<blockquote>
<p>For more details, in particular filter generation customization, run <code>memthol --filter_gen help</code>.</p>
</blockquote>
<p>If we relaunch the example without <code>--filter_gen none</code></p>
<pre><code class="language-shell-session">&#10095; memthol rsc/dumps/ctf/mini_ae.ctf
|===| Starting
| url: http://localhost:7878
| target: `rsc/dumps/ctf/mini_ae.ctf`
|===|
</code></pre>
<p>we get something like this (actual colors may vary):</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/basics_pics/filter_gen.png" alt=""/></p>
<h2>Charts</h2>
<p>This section uses the same running example as the last section.</p>
<pre><code class="language-shell-session">&#10095; memthol rsc/dumps/ctf/mini_ae.ctf
|===| Starting
| url: http://localhost:7878
| target: `rsc/dumps/ctf/mini_ae.ctf`
|===|
</code></pre>
<h3>Filter Toggling</h3>
<p>The first way to interact with a chart is to (de)activate filters. Each chart has its own filter tabs allowing to toggle filters on/off.</p>
<p>From the initial settings</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/charts_pics/init.png" alt=""/></p>
<p>click on all filters but <strong>everything</strong> to toggle them off.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/charts_pics/only_everything.png" alt=""/></p>
<p>Let's create a new chart. The only kind of chart that can be constructed currently is total size over time, so click on <strong>create chart</strong> below our current, lone chart.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/charts_pics/two_charts_1.png" alt=""/></p>
<p>Deactivate <strong>everything</strong> in the second chart.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/charts_pics/two_charts_2.png" alt=""/></p>
<p>Nice. We now have the overall total size over time in the first chart, and the details for each filter in the second one.</p>
<p>Next, notice that both charts have, on the left of their title, a down (first chart) and up (second chart) arrow. This moves the charts up and down.</p>
<p>On the right of the title, we have a settings <code>...</code> buttons which is discussed <a href="https://ocamlpro.github.io/memthol/mini_tutorial/charts.html#chart-settings">below</a>. The next button collapses the chart. If we click on the <em>collapse</em>* button of the first chart, it collapses and the button turns into an <em>expand</em> button.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/charts_pics/collapsed.png" alt=""/></p>
<p>The last button in the chart header removes the chart.</p>
<h3>Chart Settings</h3>
<p>Clicking the settings <code>...</code> button in the header of any chart display its settings. (Clicking on the button again hides them.)</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/charts_pics/settings_1.png" alt=""/></p>
<p>Currently, these chart settings only allow to rename the chart and change its <strong>display mode</strong>.</p>
<h4>Display Mode</h4>
<p>In memthol, a chart can be displayed in one of three ways:</p>
<ul>
<li>normal, the one we used so far,
</li>
<li>stacked area, where the values of each filter are displayed on top of each other, and
</li>
<li>stacked area percent, same as stacked area but values are displayed as percents of the total.
</li>
</ul>
<p>Here is the second chart from our example displayed as stacked area for instance:</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/charts_pics/settings_stacked.png" alt=""/></p>
<h2>Global Settings</h2>
<p>This section uses the same running example as the last section.</p>
<pre><code class="language-shell-session">&#10095; memthol rsc/dumps/ctf/mini_ae.ctf
|===| Starting
| url: http://localhost:7878
| target: `rsc/dumps/ctf/mini_ae.ctf`
|===|
</code></pre>
<p>There is currently only one global setting: the <em>time window</em>.</p>
<h3>Time Window</h3>
<p>The <em>time window</em> global setting controls the time interval displayed by all the charts.</p>
<p>In our example,</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/global_settings_pics/init.png" alt=""/></p>
<p>not much is happening before (roughly) <code>0.065</code> seconds. Let's have the time window start at that point:</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/global_settings_pics/time_window_1.png" alt=""/></p>
<p>Similar to filter edition, we can apply or cancel this change using the two buttons that appeared in the bottom-left corner of the header.</p>
<p>Saving these changes yields</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/global_settings_pics/time_window_2.png" alt=""/></p>
<p>Here is the same chart but with the time window upper-bound set at <code>0.074</code>.</p>
<p><img src="https://ocamlpro.github.io/memthol/mini_tutorial/global_settings_pics/time_window_3.png" alt=""/></p>
<h2>Callstack Filters</h2>
<p>Callstack filters are filters operating over allocation properties that are sequences of strings (potentially with some other data). Currently, this means <strong>allocation callstacks</strong>, where the strings are file names with line/column information.</p>
<h3>String Filters</h3>
<p>A string filter can have three shapes: an actual <em>string value</em>, a <em>regex</em>, or a <em>match anything</em> / <em>wildcard</em> filter represented by the string <code>&quot;...&quot;</code>. This wildcard filter is discussed in <a href="https://ocamlpro.github.io/memthol/mini_tutorial/callstack_filters.html#the-wildcard-filter">its own section</a> below.</p>
<p>A string value is simply given as a value. To match precisely the string <code>&quot;file_name&quot;</code>, one only needs to write <code>file_name</code>. So, a filter that matches precisely the list of strings <code>[ &quot;file_name_1&quot;, &quot;file_name_2&quot; ]</code> will be written</p>
<table>
<thead>
<tr>
<th>
</th>
<th>
</th>
<th>
</th>
</tr>
</thead>
<tbody>
<tr>
<td>string list</td>
<td>contains</td>
<td>`[ file_name_1 file_name_2 ]`</td>
</tr>
</tbody>
</table>
<p>A <em>regex</em> on the other hand has to be written between <code>#&quot;</code> and <code>&quot;#</code>. If we want the same filter as above, but want to relax the first string description to be <code>file_name_&lt;i&gt;</code> where <code>&lt;i&gt;</code> is a single digit, we write the filter as</p>
<table>
<thead>
<tr>
<th>
</th>
<th>
</th>
<th>
</th>
</tr>
</thead>
<tbody>
<tr>
<td>string list</td>
<td>contains</td>
<td>`[ #&quot;file_name_[0-9]&quot;# file_name_2 ]`</td>
</tr>
</tbody>
</table>
<h3>The Wildcard Filter</h3>
<p>The wildcard filter, written <code>...</code>, <strong>lazily</strong> (in general, see below) matches a repetition of any string-like element of the list. To break this definition down, let us separate two cases: the first one is when <code>...</code> is not followed by another string-like filter, and second one is when it is followed by another filter.</p>
<p>In the first case, <code>...</code> simply matches everything. Consider for instance the filter</p>
<table>
<thead>
<tr>
<th>
</th>
<th>
</th>
<th>
</th>
</tr>
</thead>
<tbody>
<tr>
<td>string list</td>
<td>contain</td>
<td>`[ #&quot;file_name_[0-9]&quot;# ... ]`</td>
</tr>
</tbody>
</table>
<p>This filter matches any list of strings that starts with a string accepted by the first regex filter. The following lists of strings are all accepted by the filter above.</p>
<ul>
<li><code>[ file_name_0 ]</code>
</li>
<li><code>[ file_name_7 anything at all ]</code>
</li>
<li><code>[ file_name_3 file_name_7 ]</code>
</li>
</ul>
<p>Now, there is one case when <code>...</code> is not actually lazy: when the <code>n</code> string-filters <em>after</em> it are not <code>...</code>. In this case, all elements of the list but the <code>n</code> last ones will be skipped, leaving them for the <code>n</code> last string filters.</p>
<p>For this reason</p>
<table>
<thead>
<tr>
<th>
</th>
<th>
</th>
<th>
</th>
</tr>
</thead>
<tbody>
<tr>
<td>string list</td>
<td>contain</td>
<td>`[ &hellip; #&quot;file_name_[0-9]&quot;# ]`</td>
</tr>
</tbody>
</table>
<p>does work as expected. For example, on the string list</p>
<pre><code class="language-shell-session">[ &quot;some_file_name&quot; &quot;file_name_7&quot; &quot;another_file_name&quot; &quot;file_name_0&quot; ]
</code></pre>
<p>a lazy behavior would not match. First, <code>...</code> would match anything up to and excluding a string recognized by <code>#&quot;file_name_[0-9]&quot;#</code>. So <code>...</code> would match <code>some_file_name</code>, but that's it since <code>file_name_7</code> is a match for <code>#&quot;file_name_[0-9]&quot;#</code>. Hence the filter would reject this list of strings, because there should be nothing left after the match for <code>#&quot;file_name_[0-9]&quot;#</code>. But there are still <code>another_file_name</code> and <code>file_name_0</code> left.</p>
<p>Instead, the filter works as expected. <code>...</code> discards all elements but the last one <code>file_name_0</code>, which is accepted by <code>#&quot;file_name_[0-9]&quot;#</code>.</p>
<h3>Callstack (Location) Filters</h3>
<p>Allocation callstack information is a list of tuples containing:</p>
<ul>
<li>the name of the file,
</li>
<li>the line in the file,
</li>
<li>a column range.
</li>
</ul>
<p>Currently, the range information is ignored. The line in the file is not, and one can specify a line constraint while writing a callstack filter. The <em>normal</em> syntax is</p>
<pre><code class="language-shell-session">&lt;string-filter&gt;:&lt;line-filter&gt;
</code></pre>
<p>Now, a line filter has two basic shapes</p>
<ul>
<li><code>_</code>: anything,
</li>
<li><code>&lt;number&gt;</code>: an actual value.
</li>
</ul>
<p>It can also be a range:</p>
<ul>
<li><code>[&lt;basic-line-filter&gt;, &lt;basic-line-filter&gt;]</code>: a potentially open range.
</li>
</ul>
<h4>Line Filter Examples</h4>
<table>
<thead>
<tr>
<th>
</th>
<th>
</th>
</tr>
</thead>
<tbody>
<tr>
<td>`_`</td>
<td>matches any line at all</td>
</tr>
<tr>
<td>`7`</td>
<td>matches line 7</td>
</tr>
<tr>
<td>`[50, 102]`</td>
<td>matches any line between `50` and `102`</td>
</tr>
<tr>
<td>`[50, _]`</td>
<td>matches any line greater than `50`</td>
</tr>
<tr>
<td>`[_, 102]`</td>
<td>matches any line less than `102`</td>
</tr>
<tr>
<td>`[_, _]`</td>
<td>same as `_` (matches any line)</td>
</tr>
</tbody>
</table>
<h4>Callstack Filter Examples</h4>
<p>Whitespaces are inserted for readability but are not needed:</p>
<table>
<thead>
<tr>
<th>
</th>
<th>
</th>
</tr>
</thead>
<tbody>
<tr>
<td>`src/main.ml : _`</td>
<td>matches any line of `src/main.ml`</td>
</tr>
<tr>
<td>`#&quot;.*/main.ml&quot;# : 107`</td>
<td>matches line 107 of any `main.ml` file regardless of its path</td>
</tr>
</tbody>
</table>

