---
layout: post
title: "Graphviz test"
date: 2013-10-16 14:11
comments: true
categories: viz test
---

### a test graph
{% graphviz %}
digraph G {
  compound=true;
  subgraph cluster0 {
  a -> b;
  a -> c;
  b -> d;
  c -> d;
  }
  subgraph cluster1 {
  e -> g;
  e -> f;
  }
  b -> f [lhead=cluster1];
  d -> e;
  c -> g [ltail=cluster0, lhead=cluster1];
  c -> e [ltail=cluster0];
  d -> h;
}
{% endgraphviz %}
