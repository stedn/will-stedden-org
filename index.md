---
layout: default
---

  <header class="intro" id="intro-link">
    <div class="container">
      <div class="row text-center">
        <div class="col-lg-2 col-md-12 col-sm-12 col-xs-12 order-1">
          <img class="img-fluid mx-auto d-block" height="10px;" src="/camping.png" alt="">
        </div>
        <div class="col-lg-10 col-md-12 col-sm-12 col-xs-12 order-2 align-self-center">
        <h5>Working on lots of things in parallel. Currently <a href="https://anthem.ai">machine learning in healthcare</a>, <a href="https://worcfoods.com/">a worker-owned robotics cooperative</a>, and <a href="https://viewfoil.bonkerfield.org/">personal online transparency</a>.</h5>
        </div>
      </div>
      <div class="row text-center">
        <div class="col-lg-12 col-md-12 col-sm-12 col-xs-12 order-2 align-self-center">
          <h4><a href="/timeline.html">←<small style="font-size:14px;">Timeline Resumé </small></a>Visual Resumé <a href="/classic.html"><small style="font-size:14px;">Classic Resumé</small>→</a></h4>
          <a href="https://bonkerfield.org/2015/01/a-better-linkedin/"><span class="mytooltip" tlite="se" title="<p style=&quot;margin-bottom:0;font-size:1.2em;text-align:center&quot;>Displays connections between people I've worked with, <br/> projects I've worked on, and skills required for each project.<br/> Width is scale of project, height is time occupied. <br/> You can click on a project for more details. </p>" >🛈</span></a>
        </div>
      </div>
    </div>
    <div class="align-self-center" style="max-width: 800px;margin-left: auto;margin-right: auto;padding-left:5px;padding-right:5px;padding-top:20px;" >
      <p class="alignleft">People</p>
      <p class="aligncenter">Projects</p>
      <p class="alignright">Skills</p>
    </div>
    <div id="loading-bar-spinner" class="spinner"><div class="spinner-icon"></div></div>

</header>
<script src="https://d3js.org/d3.v4.min.js"></script>
<script src="//d3js.org/d3-scale-chromatic.v0.3.min.js"></script>

<script src="sankey.js"></script>
<script>

var units = "Widgets";

// set the dimensions and margins of the graph
var maxwidth = 800
var margin = {top: 0, right: 10, bottom: 10, left: 10};
var width = document.body.clientWidth - margin.left*2 - margin.right*2;
width = width < maxwidth ? width : maxwidth
height_factor = 15*width / 800
var height = document.body.clientHeight*height_factor - margin.top - margin.bottom;
width_scale = width
var format_date = d3.timeParse("%m/%d/%y");

// format variables
var formatNumber = d3.format(",.0f"),    // zero decimal places
    format = function(d) { return formatNumber(d) + " " + units; },
    color = d3.scaleOrdinal(d3.schemePastel1);

// append the svg object to the body of the page
var svg = d3.select("body").append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform",
          "translate(" + margin.left + "," + margin.top + ")");

// Set the sankey diagram properties
var sankey = d3.sankey()
    .nodeWidth(36)
    .nodePadding(40)
    .size([width, height]);

var path = sankey.link();
d3.json("https://spreadsheets.google.com/feeds/list/1WmkDCeImSR7k5JnNn7HE4ZiDH3j4X-iIarkBrbbPvQ8/2/public/values?alt=json", function (meta_result) {
  var metadata = {};
  for (var i = 0; i < meta_result.feed.entry.length; i += 1) {
      proj = meta_result.feed.entry[i].gsx$project.$t
      metadata[proj] = {
          "color": meta_result.feed.entry[i].gsx$color.$t,
          "link": meta_result.feed.entry[i].gsx$link.$t,
          "order": meta_result.feed.entry[i].gsx$order.$t,
          "description": meta_result.feed.entry[i].gsx$description.$t
      }
  }
  d3.json("https://spreadsheets.google.com/feeds/list/1WmkDCeImSR7k5JnNn7HE4ZiDH3j4X-iIarkBrbbPvQ8/1/public/values?alt=json", function (agg_result) {
    unique_projects = {};
    for (var i = 0; i < agg_result.feed.entry.length; i += 1) {
      valval = agg_result.feed.entry[i].gsx$value.$t;
      project_val = agg_result.feed.entry[i].gsx$project.$t;
      date_val = agg_result.feed.entry[i].gsx$date.$t;
      date_val = format_date(date_val);
      if (!(project_val in unique_projects)){
        unique_projects[project_val] = {"months":0, "val":0, "date": format_date('01/01/01'), "min_date": format_date('01/01/50')};
      }
      if(parseFloat(valval)>0.025){
        unique_projects[project_val]["val"]+=parseFloat(valval);
        unique_projects[project_val]["months"]+=1;
        if (date_val > unique_projects[project_val]["date"]){
          unique_projects[project_val]["date"] = date_val
        }
        if (date_val < unique_projects[project_val]["min_date"]){
          unique_projects[project_val]["min_date"] = date_val
        }
      }
    }
    max_date = format_date('01/01/01')
    min_date = format_date('01/01/50')
    for (var k in unique_projects){
      date_val = unique_projects[k]["date"]
      if (date_val > max_date){
        max_date = date_val
      }
      if (date_val < min_date){
        min_date = date_val
      }
    }

    timescaler = d3.scaleTime().domain([min_date, max_date]).range([1,0])
// load the data
    d3.json("https://spreadsheets.google.com/feeds/list/1WmkDCeImSR7k5JnNn7HE4ZiDH3j4X-iIarkBrbbPvQ8/3/public/values?alt=json", function(result) {
      data = []
      for (var i = 0; i < result.feed.entry.length; i += 1) {
              data.push({
                  "source": result.feed.entry[i].gsx$source.$t,
                  "target": result.feed.entry[i].gsx$target.$t
              });
          }


      //set up graph in same style as original example but empty
      graph = {"nodes" : [], "links" : []};

      data.forEach(function (d) {
        graph.nodes.push({ "name": d.source});
        graph.nodes.push({ "name": d.target});
        graph.links.push({ "source": d.source,
                           "target": d.target,
                           "value": +1 });
       });

      // return only the distinct / unique nodes
      graph.nodes = d3.keys(d3.nest()
        .key(function (d) { return d.name; })
        .object(graph.nodes));

      // loop through each link replacing the text with its index from node
      graph.links.forEach(function (d, i) {
        graph.links[i].source = graph.nodes.indexOf(graph.links[i].source);
        graph.links[i].target = graph.nodes.indexOf(graph.links[i].target);
      });

      // now loop through each nodes to make nodes an array of objects
      // rather than an array of strings
      graph.nodes.forEach(function (d, i) {
        thecolor = color(d.replace(/ .*/, ""))
        theorder = 2
        theheight =  1;
        thewidth = 1;
        thedate = timescaler(max_date)
        the_date_range = ""
        thelink = ""
        if (d in metadata){
          thecolor =  metadata[d]['color'];
          theorder = metadata[d]['order'];
          thelink = metadata[d]['link']
        }
        if (d in unique_projects){
          theheight =  unique_projects[d]['months'];
          thewidth =  unique_projects[d]['val'];
          thedate =  timescaler(unique_projects[d]['date']);
          if(unique_projects[d]['min_date'].getYear()!=unique_projects[d]['date'].getYear()){
            the_date_range = (1900+unique_projects[d]['min_date'].getYear()) + "-" + (1900+unique_projects[d]['date'].getYear());
          }else{
            the_date_range = (1900+unique_projects[d]['min_date'].getYear())
          }
        }
        graph.nodes[i] = { "name": d, "color": thecolor, "order": theorder, "myheight": theheight, "mywidth": thewidth/theheight, "mydate": thedate, "link": thelink, "daterange": the_date_range  };
      });

      sankey
          .nodes(graph.nodes)
          .links(graph.links)
          .layout(32);

      left_bound = width / 8
      right_bound = 7 * width / 8

      graph.nodes.forEach(function(d) {
        d.y = d.x < right_bound && d.x > left_bound ? d.mydate*2700*height_factor/18 : d.y
        d.dy = d.x < right_bound && d.x > left_bound ? d.myheight*20*height_factor/18 : d.dy
        d.dx = d.x < right_bound && d.x > left_bound ? d.mywidth*width_scale/4.5 : d.dx
        d.x = d.x + width_scale/10*(d.x < right_bound && d.x > left_bound ? 7*(0.4-d.mywidth) : 0)
        // d.dx = d.dx*(1+(d.x < 3* width / 4 && d.x >  width / 4 ? d.order : 0))
      });
      sankey.relayout();
      // add in the links
      var link = svg.append("g").selectAll(".link")
          .data(graph.links)
        .enter().append("path")
          .attr("class", "link")
          .attr("d", path)
          .style('opacity', 0.5)
          .style("stroke-width", 2)
          .sort(function(a, b) { return b.dy - a.dy; });

      // add the link titles
      link.append("title")
            .text(function(d) {
            return d.source.name + " → " +
                    d.target.name; });

      // add in the nodes
      var node = svg.append("g").selectAll(".node")
          .data(graph.nodes)
        .enter().append("g")
          .attr("class", "node")
          .attr("transform", function(d) {
          return "translate(" + d.x + "," + d.y + ")"; });

      // add the rectangles for the nodes
      noderects = node.append("a")
          .attr("xlink:href", function(d) { return d.link; })
          .append("rect")
          .attr("height", function(d) { return d.dy; })
          .attr("width", function(d) { return d.dx; })
          .style("fill", function(d) { return d.color; })
          .style("stroke", function(d) {
          return d3.rgb(d.color).darker(2); });

      node
        .on('mouseover', function (d) {
          // Highlight the nodes: every node is green except of him
          tohilight = []
          for (var l in graph.links){
            link_d = graph.links[l]
            if (link_d.target.name === d.name){
              tohilight.push(link_d.source.name)
            }
            if (link_d.source.name === d.name){
              tohilight.push(link_d.target.name)
            }
          }
          node.select('text').style('opacity', 0.1)
          node.select('rect').style('opacity', function(d){ return tohilight.includes(d.name) ? 1 : 0.1})
          node.select('text').style('opacity', function(d){ return tohilight.includes(d.name) ? 1 : 0.1})
          d3.select(this).select('rect').style('opacity', 1)
          d3.select(this).select('text').style('opacity', 1)
          // Highlight the connections
          link
            .style('opacity', function (link_d) { return link_d.source.name === d.name || link_d.target.name === d.name ? 1 : 0.2;})
        })
        .on('mouseout', function (d) {
          node.select('rect').style('opacity', 0.9)
          node.select('text').style('opacity', 0.5)
          link.style('opacity', 0.5)
        })

      // add in the title for the nodes
      font_size = "12px"
      if(width_scale<600){
        font_size = "10px"
      }
      node.append("text")
          .attr("font-size", font_size)
          .attr("x", function(d) { return sankey.nodeWidth(); })
          .attr("y", function(d) { return 15+ d.dy ; })
          .attr("text-anchor", "end")
          .attr("transform", null)
          .text(function(d) { return d.name; })
          .attr("fill", "gray")
          .attr("opacity", "0.5")
        .filter(function(d) { return d.x < right_bound; })
          .attr("fill", "black")
          .attr("x", -5)
          .attr("y", 15)
          .attr("text-anchor", "end")
          .attr("transform", "rotate(-90)")
        .filter(function(d) { return d.x <  left_bound; })
          .attr("fill", "gray")
          .attr("y", function(d) { return 15+ d.dy ; })
          .attr("x", 0)
          .attr("transform", null)
          .attr("text-anchor", "start");
      node.append("text")
        .filter(function(d) { return d.dx > 30; })
        .filter(function(d) { return d.x < right_bound; })
        .filter(function(d) { return d.x >  left_bound; })
          .attr("font-size", font_size)
          .text(function(d) { return d.daterange; })
          .attr("opacity", "0.1")
          .attr("fill", "black")
          .attr("x", -5)
          .attr("y", 30)
          .attr("text-anchor", "end")
          .attr("transform", "rotate(-90)")

      document.getElementById("loading-bar-spinner").style.display = "none";

    });

  });
});
</script>
