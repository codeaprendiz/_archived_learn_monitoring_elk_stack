Stats | Visualization | Calculation | Action Needed
--- | --- | --- | ---
Nodes | ![](.ReadMe_images/nodes.png) | [Nodes](#Nodes) | Write action here



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