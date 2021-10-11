---
layout: graph
---
<!-- Create a div where the graph will take place -->
<div id="my_dataviz"></div>

<script>

chart();


function chart() {

extra_right = 100
  var margin = {top: 60, right: 100+extra_right, bottom: 50, left: 100},
    width = document.body.clientWidth - margin.left - margin.right,
    height = 400 - margin.top - margin.bottom;

// append the svg object to the body of the page
var svg = d3.select("#my_dataviz")
  .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform",
          "translate(" + margin.left + "," + margin.top + ")");

parseDate = d3.time.format("%Y-%m").parse

fetch("https://docs.google.com/spreadsheets/d/1Yp7CPoeeuPLLBhxKRa4SIzQBT_PXXti8uOfKhQfmndY/gviz/tq?tqx=out:json")
    .then(res => res.text())
    .then(text => {
    meta_result = JSON.parse(text.substr(47).slice(0, -2))

    keys = []
    for (var i = 0; i < 14; i += 1){
      keys.push(meta_result.table.cols[i].label)
    }
    var data = [];
    for (var i = 0; i < meta_result.table.rows.length; i += 1) {
      to_push = {};
      for (var j = 0; j < 14; j += 1){
        if (j == 0){
          to_push[keys[j]]=meta_result.table.rows[i].c[j].f
        } else{
          to_push[keys[j]]=meta_result.table.rows[i].c[j].v
        }
      }
      data.push(to_push)
    }

    max_income = 20000
    //////////
    // GENERAL //
    //////////
    keys = keys.slice(1)
    // List of groups = header of the csv files
    // var keys = data.columns.slice(1)

    data.forEach(function(d,i) {
      d.month = parseDate(d.month);
    });
    // color palette
    var color = d3.scaleOrdinal()
      .domain(keys)
      .range(d3.schemeSet3);

    //stack the data?
    var stackedData = d3.stack()
      .keys(keys)
      (data)

    //////////
    // AXIS //
    //////////

    // Add X axis
    var x = d3.time.scale()
      .domain(d3.extent(data, function(d) { return d.month; }))
      .range([ 0, width ]);
    var xAxis = svg.append("g")
      .attr("class","axis")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(x).ticks(5))

    // Add X axis label:
    svg.append("text")
        .attr("class","axis")
        .attr("text-anchor", "end")
        .attr("x", width/2)
        .attr("y", height+40 )
        .text("Time");

    // Add Y axis label:
    svg.append("text")
      .attr("class","axis")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left*0.75)
      .attr("x",0 - (height / 2))
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Quarterly Expenses");

    // Add Y axis
    var y = d3.scaleLinear()
      .domain([0, max_income+100])
      .range([ height, 0 ]);
    svg.append("g")
      .attr("class","axis")
      .call(d3.axisLeft(y).ticks(5))



    //////////
    // BRUSHING AND CHART //
    //////////

    // Add a clipPath: everything out of this area won't be drawn.
    var clip = svg.append("defs").append("svg:clipPath")
        .attr("id", "clip")
        .append("svg:rect")
        .attr("width", width )
        .attr("height", height )
        .attr("x", 0)
        .attr("y", 0);

    // Add brushing
    var brush = d3.brushX()                 // Add the brush feature using the d3.brush function
        .extent( [ [0,0], [width,height] ] ) // initialise the brush area: start at 0,0 and finishes at width,height: it means I select the whole graph area
        .on("end", updateChart) // Each time the brush selection changes, trigger the 'updateChart' function

    // Create the scatter variable: where both the circles and the brush take place
    var areaChart = svg.append('g')
      .attr("clip-path", "url(#clip)")

    // Area generator
    var area = d3.area()
      .x(function(d) { return x(d.data.month); })
      .y0(function(d) { return y(d[0]); })
      .y1(function(d) { return y(d[1]); })

    // Show the areas
    areaChart
      .selectAll("mylayers")
      .data(stackedData)
      .enter()
      .append("path")
        .attr("class", function(d) { return "myArea " + d.key })
        .style("fill", function(d) { return color(d.key); })
        .attr("d", area)

    // Add the brushing
    areaChart
      .append("g")
        .attr("class", "brush")
        .call(brush);

    var valueline = d3.line()
        .x(function(d) { return x(d.month); })
        .y(function(d) { return y(d.income); });

    var incomeline = svg.append('g')
        .attr("clip-path", "url(#clip)")

    // incomeline.append("path")
    //   .data([income_data])
    //   .attr("class", "line")
    //   .attr("d", valueline);

    var idleTimeout
    function idled() { idleTimeout = null; }

    // A function that update the chart for given boundaries
    function updateChart() {

      extent = d3.event.selection

      // If no selection, back to initial coordinate. Otherwise, update X axis domain
      if(!extent){
        if (!idleTimeout) return idleTimeout = setTimeout(idled, 350); // This allows to wait a little bit
        x.domain(d3.extent(data, function(d) { return d.month; }))
      }else{
        x.domain([ x.invert(extent[0]), x.invert(extent[1]) ])
        areaChart.select(".brush").call(brush.move, null) // This remove the grey brush area as soon as the selection has been done
      }

      // Update axis and area position
      xAxis.transition().duration(1000).call(d3.axisBottom(x).ticks(5))
      areaChart
        .selectAll("path")
        .transition().duration(1000)
        .attr("d", area)
      // incomeline
      //   .selectAll("path")
      //   .transition().duration(1000)
      //   .attr("d", valueline);

      }



      //////////
      // HIGHLIGHT GROUP //
      //////////

      // What to do when one group is hovered
      var highlight = function(d){
        console.log(d)
        // reduce opacity of all groups
        d3.selectAll(".myArea").style("opacity", .1)
        // expect the one that is hovered
        d3.select("."+d).style("opacity", 1)
      }

      // And when it is not hovered anymore
      var noHighlight = function(d){
        d3.selectAll(".myArea").style("opacity", 1)
      }



      //////////
      // LEGEND //
      //////////

      // Add one dot in the legend for each name.
      var size = 20
      keys = keys.reverse()
      svg.selectAll("myrect")
        .data(keys)
        .enter()
        .append("rect")
          .attr("x", width+extra_right/2)
          .attr("y", function(d,i){ return 10 + i*(size+5)}) // 100 is where the first dot appears. 25 is the distance between dots
          .attr("width", size)
          .attr("height", size)
          .style("fill", function(d){ return color(d)})
          .on("mouseover", highlight)
          .on("mouseleave", noHighlight)

      // Add one dot in the legend for each name.
      svg.selectAll("mylabels")
        .data(keys)
        .enter()
        .append("text")
          .attr("x", width +extra_right/2+ size*1.2)
          .attr("y", function(d,i){ return 10 + i*(size+5) + (size/2)}) // 100 is where the first dot appears. 25 is the distance between dots
          .style("fill", function(d){ return color(d)})
          .text(function(d){ return d})
          .attr("text-anchor", "left")
          .style("alignment-baseline", "middle")
          .on("mouseover", highlight)
          .on("mouseleave", noHighlight)

  })
}
</script>