---
layout: default
---

  <header class="intro" id="intro-link">
    <div class="container">
      {% include bio.html %}
      <!-- current projects -->
      <div class="row text-center">
        <div class="col-lg-12 col-md-12 col-sm-12 col-xs-12 order-2 align-self-center" style="margin-top:25px;">
          <h4>Current Projects</h4>
        </div>
      </div>
      <div class="row" style="margin: 0 auto;max-width:600px;">
        <div class="column">
          <a class="mytooltip" tlite="se" title="<p style=&quot;margin-bottom:0;font-size:1.2em;text-align:center&quot;>Solarpunk Travel Cooperative - reimagining the future of travel</p>" href="https://trailcoop.org"><img src="images/solarpunk_travel.png" style="width:100%"></a>
        </div>
        <div class="column">
          <a class="mytooltip" tlite="se" title="<p style=&quot;margin-bottom:0;font-size:1.2em;text-align:center&quot;>solarpunk.blue - to imagine a more equitable and sustainable world</p>" href="https://solarpunk.blue"><img src="images/solarpunk_blue_logo.jpeg" style="width:100%"></a>
        </div>
        <div class="column">
          <a class="mytooltip" tlite="se" title="<p style=&quot;margin-bottom:0;font-size:1.2em;text-align:center&quot;>Walkable - a board game about tranforming our cities and towns for a better future</p>" href="#"><img  src="images/walkable.png" style="width:100%"></a>
        </div>
      </div>
      <!-- resume -->
      <div class="row text-center" style="margin-top:50px;">
        <div class="col-lg-12 col-md-12 col-sm-12 col-xs-12 order-2 align-self-center">
          <h4>Project Timeline
          <a href="https://bonkerfield.org/2020/05/timeline-streamgraph-google-sheet/"><span class="mytooltip" tlite="se" title="<p style=&quot;margin-bottom:0;font-size:1.2em;text-align:center&quot;>Displays concurrent projects I've worked on. <br/> Width indicates how much time I spent on each project. <br/> Time runs vertical with most recent work at the top.</p>" >🛈</span></a></h4>
        </div>
      </div>
    </div>

</header>

<div id="loading-bar-spinner" class="spinner"><div class="spinner-icon"></div></div>
<div class="chart" id="chart">
</div>
<script src="https://d3js.org/d3.v3.js"></script>

<script>

chart();

// window.addEventListener("resize", chart);


function chart() {

  var format = d3.time.format("%m/%d/%y");

  var margin = {top: 20, right: 40, bottom: 30, left: 30};

  var width = document.body.clientWidth - margin.left - margin.right;
  var height = document.body.clientHeight*20 - margin.top - margin.bottom;

  var tooltip = d3.select(".chart")
      .append("div")
      .attr("class", "remove")
      .style("position", "absolute")
      .style("z-index", "20")
      .style("visibility", "hidden");

  var x = d3.scale.linear()
      .range([0, width]);

  var y = d3.time.scale()
      .range([height-10, 0]);

  var yAxis = d3.svg.axis()
      .scale(y);

  var stack = d3.layout.stack()
      .offset("silhouette")
      .values(function(d) { return d.values; })
      .x(function(d) { return d.date; })
      .y(function(d) { return d.value; });

  var nest = d3.nest()
      .key(function(d) { return d.project; });

  var area = d3.svg.area()
      .interpolate("basis")
      .y(function(d) { return y(d.date); })
      .x0(function(d) { return x(d.y0); })
      .x1(function(d) { return x(d.y0 + d.y); });

  var svg = d3.select(".chart").append("div")
      .classed("svg-container", true) //container class to make it responsive
      .append("svg")
      .attr("preserveAspectRatio", "xMinYMin meet")
      .attr("viewBox", "0 0 "+width+" " +height)
      .classed("svg-content-responsive", true)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");


fetch("https://docs.google.com/spreadsheets/d/1WmkDCeImSR7k5JnNn7HE4ZiDH3j4X-iIarkBrbbPvQ8/gviz/tq?tqx=out:json&sheet=links")
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
    var metadata = {};
    for (var i = 0; i < meta_result.table.rows.length; i += 1) {
        proj = meta_result.table.rows[i].c[0].v
        metadata[proj] = {
            "color": meta_result.table.rows[i].c[1].v,
            "link": meta_result.table.rows[i].c[2].v,
            "order": meta_result.table.rows[i].c[3].v,
            "description": meta_result.table.rows[i].c[4].v
        }
    }

fetch("https://docs.google.com/spreadsheets/d/1WmkDCeImSR7k5JnNn7HE4ZiDH3j4X-iIarkBrbbPvQ8/gviz/tq?tqx=out:json&sheet=history")
    .then(res => res.text())
    .then(text => {
      result = JSON.parse(text.substr(47).slice(0, -2));
      var data = [];
      all_dates = {};
      unique_projects = {};
      for (var i = 0; i < result.table.rows.length; i += 1) {
          date_val = result.table.rows[i].c[2].f;
          project_val = result.table.rows[i].c[0].v;
          data.push({
              "date": date_val,
              "value": result.table.rows[i].c[1].v,
              "project": project_val
          });
          if (!(date_val in all_dates)){
            all_dates[date_val] = true;
          }
          if (!(project_val in unique_projects)){
            unique_projects[project_val] = {};
          }
          unique_projects[project_val][date_val]=true;
      }
      console.log(data)

      for (p in unique_projects){
        for (dt in all_dates) {
          if (!(dt in unique_projects[p])){
            data.push({
              "date": dt,
              "value": 0,
              "project": p
            });
          }
        }
      }


      data.forEach(function(d) {
        d.date = format.parse(d.date);
        d.value = +d.value;
        if (d.project in metadata){
          d.color = metadata[d.project]['color']
          d.link = metadata[d.project]['link']
          d.order = metadata[d.project]['order']
          d.description = metadata[d.project]['description']
        } else {
          d.color = '#111111'
          d.link = '#'
          d.order = 1
          d.description = ''
        }
      });

      function compare( a, b ) {
        if ( b.order > a.order ){
          return -1;
        }
        if ( a.order > b.order ){
          return 1;
        }
        if ( b.date > a.date ){
          return -1;
        }
        if ( a.date > b.date ){
          return 1;
        }
        return 0;
      }

      data.sort(compare);

      var layers = stack(nest.entries(data));
      console.log((layers[0].values))

      y.domain(d3.extent(data, function(d) { return d.date; }));
      x.domain([0, 1.05*d3.max(data, function(d) { return d.y0 + d.y; })]);

      svg.selectAll(".layer")
          .data(layers)
        .enter().append("a")
          .attr("xlink:href", function(d) { return d.values[0].link; })
          .append("path")
          .attr("class", "layer")
          .attr("opacity", 0.8)
          .attr("d", function(d) { return area(d.values); })
          .style("fill", function(d) { return d.values[0].color; });

      svg.selectAll(".layer").transition()
        .duration(1)
        .attr("opacity", 0.8);

      prefer_x_width = 100
      min_x_width = 50
      x_width_step = 5

      function transform_func(d){
        txt_len = d.values[0].project.length
        for (width_cutoff = prefer_x_width; width_cutoff > min_x_width; width_cutoff-=x_width_step){
          min_y = 100000
          min_xl = 0
          min_xr = 0
          for (i=0; i < d.values.length; i++){
            y_val = y(d.values[i].date)
            xl_val = x(d.values[i].y0)
            xr_val = x(d.values[i].y0+d.values[i].y)
            if ((xr_val-xl_val)>width_cutoff && y_val < min_y ){
              min_y = y_val
              min_xl = xl_val
              min_xr = xr_val
            }
          }
          min_x = (min_xl + min_xr)/2.
          if (min_x >= width_cutoff) {
            scale = (min_xr-min_xl)/Math.sqrt(txt_len)/50
            return "translate("+min_x+","+(min_y-10)+") scale("+scale+") translate(-"+(3*txt_len)+",30)"
          }
        }
      }
      function visible_func(d){
        min_y = 100000
        min_xl = 0
        min_xr = 0
        txt_len = d.values[0].project.length
        for (i=0; i < d.values.length; i++){
          y_val = y(d.values[i].date)
          xl_val = x(d.values[i].y0)
          xr_val = x(d.values[i].y0+d.values[i].y)
          if ((xr_val-xl_val)>min_x_width && y_val < min_y ){
            min_y = y_val
            min_xl = xl_val
            min_xr = xr_val
          }
        }
        return (min_xr-min_xl) < min_x_width+x_width_step ? 'hidden' : 'visible'
      }
      svg.selectAll("text")
          .data(layers)
        .enter().append("text")
        .attr('class','nohover')
        .attr("fill", '#333')
        .attr("visibility", visible_func)
        .attr("transform", transform_func)
        .text((d) => d.values[0].project);

      max_date = d3.max(data, function(d) { return d.date; });
      min_date = d3.min(data, function(d) { return d.date; });
      var dateArray = d3.time.scale()
                .domain([new Date(min_date.getFullYear(), 12, 1), new Date(max_date.getFullYear()-1, 12, 1)])
                .ticks(d3.time.years, 1);
      dateArray.push(max_date)
      const monthNames = ["January", "February", "March", "April", "May", "June",
        "July", "August", "September", "October", "November", "December"
      ];
      yAxis = yAxis.tickValues(dateArray)
        .tickFormat(d => ('<- ' + ((d==max_date) ? (monthNames[d.getMonth()]+', '+d.getFullYear()) : d.getFullYear()-1)));

      svg.append("g")
          .attr("class", "yaxis")
          .call(yAxis.orient("left"))
          .selectAll("text")
            .style("text-anchor", "end")
            .style("color", "#999")
            .attr("dx", ".4em")
            .attr("dy", "-.6em")
            .attr("transform", "rotate(-90)" );

      svg.selectAll(".layer")
        .on("mouseover", function(d, i) {
          svg.selectAll(".layer").transition()
            .duration(100)
            .attr("opacity", function(d, j) {
              return j != i ? 0.8 : 1;
        })})
        .on("mousemove", function(d, i) {

          mouse = d3.mouse(document.body);
          mousex = mouse[0];
          mousey = mouse[1];
          f = d.values[0]
          var date = f.date
          txt_width = f.description.length
          tooltip
            .style("left", (mousex-txt_width*4) +"px")
            .style("top", (mousey+100) +"px")
            .html("<div class='key'>" + f.project + "</div><div class='desc'>"+f.description+"</div>")
            .style("visibility", "visible");
        })
        .on("mouseout", function(d, i) {
          svg.selectAll(".layer").transition()
            .duration(100)
            .attr("opacity", 0.8);
          tooltip.style("visibility", "hidden");
        });

        var gradient = svg.append("defs")
          .append("linearGradient")
          .attr("id", "gradient")
          .attr("x1", "100%")
          .attr("y1", "100%")
          .attr("x2", "100%")
          .attr("y2", "0%")
          .attr("spreadMethod", "pad");

        gradient.append("stop")
          .attr("offset", "0%")
          .attr("stop-color", "#000")
          .attr("stop-opacity", 1);

        gradient.append("stop")
          .attr("offset", "100%")
          .attr("stop-color", "#000")
          .attr("stop-opacity", 0);

        svg.append("rect")
          .attr('class','nohover')
          .attr("width", 2*width)
          .attr("height", height/3)
          .attr("x", -width/2)
          .attr("y", 2*height/3)
          .style("fill", "url(#gradient)");
        document.getElementById("loading-bar-spinner").style.display = "none";
        // Add 'curtain' rectangle to hide entire graph
        var curtain = svg.append('rect')
         .attr('x', -1 * width)
         .attr('y', -1 * height)
         .attr('height', height)
         .attr('width', width)
         .attr('class', 'curtain')
         .attr('transform', 'rotate(180)')
         .style('fill', '#222')

        // Create a shared transition for anything we're animating
        var t = svg.transition()
         .delay(100)
         .duration(3000)
         .ease('linear')
         .each('end', function() {
           d3.select('line.guide')
             .transition()
             .style('opacity', 0)
             .remove()
         });

        t.select('rect.curtain')
          .attr('height', 0);
        t.select('line.guide')
          .attr('transform', 'translate(' + height + ', 0)');

    });

  });
}
</script>
