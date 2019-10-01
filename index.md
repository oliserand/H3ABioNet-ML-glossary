---
title: ""
keywords: H3ABioNet, Machine Learning, Supervised Learning, Unsupervised Learning, Classification, Regression, Clustering, Dimensionality Reduction
tags: [formating]
last_updated: July 29, 2019
author_profile: true
authors: Amel, Anmol & Shakun
hide_sidebar: true
toc: false
permalink: index.html
---

[![H3ABioNet logo](assets/images/LOGO-HEADER.jpg "H3ABioNet logo")](https://h3abionet.org)

This website aims to provide information about machine learning terms glossary within the [H3ABioNet](https://h3abionet.org/) consortium, by introducing the ML jargon and explaining it using very simple terms and including examples from the biomedical research field. We believe this Glossary will help folks from non computing background get familiar with ML terms and methods in order to enhance the use of ML to answer biological questions.

Biologists no longer rely on traditional laboratories to discover novel biomarkers for a given disease, but make use of the continuously growing genomic datasets that are publicly available to determine the biomarkers. Technologies for capturing data in biology are becoming cheaper and more effective, and this has given rise to a new era of big data in bioinformatics. These large biological datasets can be effectively analysed using machine learning aproaches.

<style>

.node circle {
  fill: #fff;
  stroke: steelblue;
  stroke-width: 3px;
}

.node text {
  font: 12px sans-serif;
}

.link {
  fill: none;
  stroke: #ccc;
  stroke-width: 2px;
}

</style>

<div id="graph"></div>

<!-- load the d3.js library -->	
<script src="https://d3js.org/d3.v4.min.js"></script>
<script>

var treeData =
  {
    "name": "machine learning",
    "children": [
      { 
        "name": "Supervised Learning",
        "children": [
          { "name": "Classification" },
          { "name": "Regression" }
        ]
      },
      { 
        "name": "Unsupervised Learning",
        "children": [
          { "name": "Clustering" },
          { "name": "Dimensionality Reduction" }
        ]
      }
    ]
  };

// Set the dimensions and margins of the diagram
var margin = {top: 20, right: 20, bottom: 100, left: 190},
    width = 1160 - margin.left - margin.right,
    height = 600 - margin.top - margin.bottom;

// append the svg object to the body of the page
// appends a 'group' element to 'svg'
// moves the 'group' element to the top left margin
var svg = d3.select("#graph").append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform", "translate("
          + margin.left + "," + margin.top + ")");
 
var i = 0,
    duration = 750,
    root;

// declares a tree layout and assigns the size
var treemap = d3.tree().size([height, width]);

// Assigns parent, children, height, depth
root = d3.hierarchy(treeData, function(d) { return d.children; });
root.x0 = height / 2;
root.y0 = 0;

// Collapse after the second level
root.children.forEach(collapse);

update(root);

// Collapse the node and all it's children
function collapse(d) {
  if(d.children) {
    d._children = d.children
    d._children.forEach(collapse)
    d.children = null
  }
}

function update(source) {

  // Assigns the x and y position for the nodes
  var treeData = treemap(root);

  // Compute the new tree layout.
  var nodes = treeData.descendants(),
      links = treeData.descendants().slice(1);

  // Normalize for fixed-depth.
  nodes.forEach(function(d){ d.y = d.depth * 180});

  // ****************** Nodes section ***************************

  // Update the nodes...
  var node = svg.selectAll('g.node')
      .data(nodes, function(d) {return d.id || (d.id = ++i); });

  // Enter any new modes at the parent's previous position.
  var nodeEnter = node.enter().append('g')
      .attr('class', 'node')
      .attr("transform", function(d) {
        return "translate(" + source.y0 + "," + source.x0 + ")";
    })
    .on('click', click);

  // Add Circle for the nodes
  nodeEnter.append('circle')
      .attr('class', 'node')
      .attr('r', 1e-6)
      .style("fill", function(d) {
          return d._children ? "lightsteelblue" : "#fff";
      });

  // Add labels for the nodes
  nodeEnter.append('text')
      .attr("dy", ".35em")
      .attr("x", function(d) {
          return d.children || d._children ? -13 : 13;
      })
      .attr("text-anchor", function(d) {
          return d.children || d._children ? "end" : "start";
      })
      .text(function(d) { return d.data.name; });

  // UPDATE
  var nodeUpdate = nodeEnter.merge(node);

  // Transition to the proper position for the node
  nodeUpdate.transition()
    .duration(duration)
    .attr("transform", function(d) { 
        return "translate(" + d.y + "," + d.x + ")";
     });

  // Update the node attributes and style
  nodeUpdate.select('circle.node')
    .attr('r', 10)
    .style("fill", function(d) {
        return d._children ? "lightsteelblue" : "#fff";
    })
    .attr('cursor', 'pointer');


  // Remove any exiting nodes
  var nodeExit = node.exit().transition()
      .duration(duration)
      .attr("transform", function(d) {
          return "translate(" + source.y + "," + source.x + ")";
      })
      .remove();

  // On exit reduce the node circles size to 0
  nodeExit.select('circle')
    .attr('r', 1e-6);

  // On exit reduce the opacity of text labels
  nodeExit.select('text')
    .style('fill-opacity', 1e-6);

  // ****************** links section ***************************

  // Update the links...
  var link = svg.selectAll('path.link')
      .data(links, function(d) { return d.id; });

  // Enter any new links at the parent's previous position.
  var linkEnter = link.enter().insert('path', "g")
      .attr("class", "link")
      .attr('d', function(d){
        var o = {x: source.x0, y: source.y0}
        return diagonal(o, o)
      });

  // UPDATE
  var linkUpdate = linkEnter.merge(link);

  // Transition back to the parent element position
  linkUpdate.transition()
      .duration(duration)
      .attr('d', function(d){ return diagonal(d, d.parent) });

  // Remove any exiting links
  var linkExit = link.exit().transition()
      .duration(duration)
      .attr('d', function(d) {
        var o = {x: source.x, y: source.y}
        return diagonal(o, o)
      })
      .remove();

  // Store the old positions for transition.
  nodes.forEach(function(d){
    d.x0 = d.x;
    d.y0 = d.y;
  });

  // Creates a curved (diagonal) path from parent to the child nodes
  function diagonal(s, d) {

    path = `M ${s.y} ${s.x}
            C ${(s.y + d.y) / 2} ${s.x},
              ${(s.y + d.y) / 2} ${d.x},
              ${d.y} ${d.x}`

    return path
  }

  // Toggle children on click.
  function click(d) {
    if (d.children) {
        d._children = d.children;
        d.children = null;
      } else {
        d.children = d._children;
        d._children = null;
      }
    update(d);
  }
}

</script>


* [Introduction to Machine Learning]({{ site.baseurl}}{% link pages/introduction_to_ml.md %})
* [Supervised Learning]({{ site.baseurl}}{% link pages/categories/supervised/supervised_learning.md %})
* [Unsupervised Learning]({{ site.baseurl}}{% link pages/categories/unsupervised/unsupervised_learning.md %})
* [Data preprocessing]({{ site.baseurl}}{% link pages/genomics_analysis/RNA-Seq/RNA-Seq-1-0.md %})




You can use the buttons at the top of the page to navigate through the Glossary different terms, or the side panel to access the subsections within each.

This ML glossary repo was [forked from this H3ABioNet-SOPs repo](https://github.com/h3abionet/H3ABionet-SOPs). Contributions are welcome! To do so, open an issue or a pull request. Additionally, check out our [contribution guide]({{ site.baseurl}}{% link pages/contributing/cont_tech-guide-1.md %}).


### Funding
The H3ABioNet Machine Learning Glossary was developped by the H3ABioNet Machine Learning and Big Data project members. The development of  is supported by the H3Africa program grant U24HG006941 from the National Human Genome Research Institute (NHGRI) of the National Institutes of Health (NIH) entitled “H3ABioNet: Informatics Solutions for H3Africa”. The content is solely the responsibility of the authors and does not necessarily represent the official views of the National Institutes of Health.

### References


[//]: <> (These are common abbreviations in the page.)
*[ML]: Machine Learning
*[H3Africa]: Human Heredity and Health in Afrcia Consortium
*[H3ABioNet]: The Bioinformatics Network within the H3Africa Consortium
