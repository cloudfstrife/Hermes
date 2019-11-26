---
title: "Graphviz绘图（五）：绘制一张时序图"
date: 2019-11-26T12:30:55+08:00
categories:
- graphviz
tags:
- graphviz
- "Sequence Diagram"
keywords:
- graphviz
- "Sequence Diagram"
---

使用Graphviz画一张时序图

<!--more-->

```
digraph sequence{
	graph [rankdir="LR";splines="polyline"]
	
	subgraph client {
		graph [rank="same"]
		node[shape="box";width=0.1;height=0.2;label=""]
		edge[dir="none";style="dashed"]
		
		client_start[shape="plaintext";label="client"]
		client_end [shape="point";width=0;height=0]
		
		client_start->client_01->client_02->client_end
	}
	
	subgraph proxy {
		graph [rank="same"]
		node[shape="box";width=0.1;height=0.2;label=""]
		edge[dir="none";style="dashed"]
		
		proxy_start[shape="plaintext";label="proxy"]
		proxy_end [shape="point";width=0;height=0]
		
		proxy_start->proxy_01->proxy_02->proxy_end
	}
	
	
	subgraph gateway {
		graph [rank="same"]
		node[shape="box";width=0.1;height=0.2;label=""]
		edge[dir="none";style="dashed"]
		
		gateway_start[shape="plaintext";label="gateway"]
		gateway_end [shape="point";width=0;height=0]
		
		gateway_start->gateway_01->gateway_02->gateway_end
	}
	
	subgraph service {
		graph [rank="same";]
		edge[dir="none";style="dashed"]
		node[shape="box";width=0.1;height=0.7;label=""]
		
		service_start[shape="plaintext";label="service"]
		service_end [shape="point";width=0;height=0]
		
		service_start->service_01 ->service_end
	}
	
	client_01->proxy_01 [label="send request"]
	proxy_01->gateway_01 [label="redirect request"]
	gateway_01->service_01:nw [label="redirect request"]
	service_01:sw->gateway_02[label="send response"]
	gateway_02->proxy_02[label="redirect response"]
	proxy_02->client_02[label="redirect response"]
}

```

Build

```
dot -T svg -o Testing.svg Testing.dot
```

![001](/images/other/graphviz/5/001.svg)