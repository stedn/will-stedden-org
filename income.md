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


fetch("https://docs.google.com/spreadsheets/d/1Yp7CPoeeuPLLBhxKRa4SIzQBT_PXXti8uOfKhQfmndY/gviz/tq?tqx=out:json&sheet=Income")
    .then(res => res.text())
    .then(text => {
    meta_result = JSON.parse(text.substr(47).slice(0, -2))

  var income_data = [];
  for (var i = 0; i < meta_result.table.rows.length; i += 1) {
      income_data.push({
          "month": meta_result.table.rows[i].c[0].f,
          "income": meta_result.table.rows[i].c[2].v,
          "expenses": meta_result.table.rows[i].c[1].v,
      })
  }

  income_data.forEach(function(d) {
      d.month = parseDate(d.month);
      d.income = +d.income;
      d.expenses = +d.expenses;
  });
  var max_income = d3.max(income_data, function(d) { return +d.income;} );
  console.log('income_data')
    //////////
    // AXIS //
    //////////

    // Add X axis
    var x = d3.time.scale()
      .domain(d3.extent(income_data, function(d) { return d.month; }))
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
      .text("Quarterly Income/Expenses");

    // Add Y axis
    var y = d3.scaleLinear()
      .domain([0, max_income+100])
      .range([ height, 0 ]);
    svg.append("g")
      .attr("class","axis")
      .call(d3.axisLeft(y).ticks(5))

    var size = 20

    svg.append("text")
        .attr("x", width + extra_right/3)
        .attr("y", 10 + 1*(size+5) + (size/2)) // 100 is where the
        .style("fill", "steelblue")
        .text('Income Post-Tax')
        .attr("text-anchor", "left")
        .style("alignment-baseline", "middle")

    svg.append("text")
        .attr("x", width + extra_right/3)
        .attr("y", 10 + 2*(size+5) + (size/2)) // 100 is where the
        .style("fill", "tomato")
        .text('Expenses')
        .attr("text-anchor", "left")
        .style("alignment-baseline", "middle")

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

  var incomeline_data = d3.line()
      .x(function(d) { return x(d.month); })
      .y(function(d) { return y(d.income); });

  var incomeline = svg.append('g')
      .attr("clip-path", "url(#clip)")

  incomeline.append("path")
    .data([income_data])
    .attr("class", "income")
    .attr("d", incomeline_data);

  var expenseline_line = d3.line()
      .x(function(d) { return x(d.month); })
      .y(function(d) { return y(d.expenses); });

  var expenseline = svg.append('g')
      .attr("clip-path", "url(#clip)")

  expenseline.append("path")
    .data([income_data])
    .attr("class", "expense")
    .attr("d", expenseline_line);


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
      }

      // Update axis and area position
      xAxis.transition().duration(1000).call(d3.axisBottom(x).ticks(5))
      incomeline
        .selectAll("path")
        .transition().duration(1000)
        .attr("d", incomeline_data);


      incomeline
        .selectAll("path")
        .transition().duration(1000)
        .attr("d", incomeline_data);
      }
  })
}
</script>