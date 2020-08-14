Stats | Visualization | Calculation | Action Needed
--- | --- | --- | ---
Nodes | ![](https://github.com/codeaprendiz/_assets/blob/master/monitoring-kitchen/metricbeat/nodes.png) | [Nodes](#Nodes) | Write action here



### Nodes

- kubernetes.node.name
  - Kubernetes node name
  - type: keyword

**Aggregation**
  
```bash
## Cardinality ##
a = Cardinality(kubernetes.node.name)

GroupBy-Everything
```