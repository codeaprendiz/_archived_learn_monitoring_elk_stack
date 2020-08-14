|               Stats                                                |           Visualization                                                                                                                          | Action Needed
|--------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------  |---------------------------- | 
|[Nodes](#Nodes)                                                     | ![](https://github.com/codeaprendiz/_assets/blob/master/monitoring-kitchen/metricbeat/nodes.png)                                                 | `kubectl get nodes`
|[Deployments](#Deployments)                                         | ![](https://github.com/codeaprendiz/_assets/blob/master/monitoring-kitchen/metricbeat/deployments.png)                                           | `kubectl get deployments`
|[Available Pods Per Deployment](#Available-pods-per-deployment)     | ![](https://github.com/codeaprendiz/_assets/blob/master/monitoring-kitchen/metricbeat/available-pods-per-deployment.png)                         | `kubectl get pods` <br>  `kubectl get deployments`
|[Desired-pods](#Desired-pods)                                       | ![](https://github.com/codeaprendiz/_assets/blob/master/monitoring-kitchen/metricbeat/desired-pods.png)                                          | `kubectl get pods`
|[Available pods](#Available-pods)                                   | ![](https://github.com/codeaprendiz/_assets/blob/master/monitoring-kitchen/metricbeat/available-pods.png)                                        | `kubectl get pods`
|[Unavailable pods](#Unavailable-pods)                               | ![](https://github.com/codeaprendiz/_assets/blob/master/monitoring-kitchen/metricbeat/unavailable-pods.png)                                      | `kubectl get pods`
|[Unavailable pods per deployment](#Unavailable-pods-per-deployment) | ![](https://github.com/codeaprendiz/_assets/blob/master/monitoring-kitchen/metricbeat/unavailable-pods-per-deployment1.png)                       | 
|[CPU Usage by node](#CPU-usage-by-node)                             | ![](https://github.com/codeaprendiz/_assets/blob/master/monitoring-kitchen/metricbeat/cpu-usage-by-node.png)                                     | 



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

### Deployments

- kubernetes.deployment.name
  - Kubernetes deployment name
  - type: keyword
  
**Aggregation**
  
```bash
## Cardinality ##
a = Cardinality(kubernetes.deployment.name)

GroupBy-Everything
``` 

### Available-pods-per-deployment

- kubernetes.deployment.replicas.available
  - Deployment available replicas
  - type: integer

- kubernetes.deployment.name
  - Kubernetes deployment name
  - type: keyword

**Aggregation**

- Time field : @timestamp
- Interval : auto

```bash
## Average ##
a = Avg(kubernetes.deployment.replicas.available)

GroupBy-Terms : kubernetes.deployment.name
Top - 10000
OrderBy - Doc Count(default)
Desc
```

### Desired-pods

- kubernetes.deployment.replicas.desired
  - Deployment number of desired replicas (spec)  
  - type: integer
   
**Aggregation**

- Data timerange mode: Last value
- Time field : @timestamp
- Inverval : auto

```bash
## Sum ##
a = Sum(kubernetes.deployment.replicas.desired)

GroupBy-Everything
```
 
### Available-pods

- kubernetes.deployment.replicas.available
  - Deployment available replicas  
  - type: integer 

**Aggregation**

- Data timerange mode: Last value
- Time field : @timestamp
- Inverval : auto

```bash
## Sum ##
a = Sum(kubernetes.deployment.replicas.available)

GroupBy-Everything
```

### Unavailable-pods

- kubernetes.deployment.replicas.unavailable
  - Deployment unavailable replicas
  - type: integer
 
**Aggregation**

- Data timerange mode: Last value
- Time field : @timestamp
- Inverval : auto

```bash
## Sum ##
a = Sum(kubernetes.deployment.replicas.unavailable)

GroupBy-Everything
``` 
 
### Unavailable-pods-per-deployment

- kubernetes.deployment.replicas.unavailable
  - Deployment unavailable replicas
  - type: integer
  
**Aggregation**

- Time field : @timestamp
- Inverval : auto

```bash
## Average ##
a = Avg(kubernetes.deployment.replicas.unavailable)

GroupBy-Terms : kubernetes.deployment.name
Top : 10000
OrderBy : Doc Count(default)
Desc
``` 

### CPU-usage-by-node


- kubernetes.container.cpu.usage.nanocores
  - CPU used nanocores  
  - type: long

**Aggregation**

- Time field : @timestamp
- Inverval : auto
- Data Formatter : Custom          
- Format string : 0.0a
- Template : {{value}} nanocores
- Chart type : Line

```bash
## Sum ##
a = Sum(kubernetes.container.cpu.usage.nanocores)

GroupBy-Terms : kubernetes.node.name
Top : 10000
OrderBy : a
Desc
``` 

- kubernetes.node.cpu.capacity.cores
  - Node CPU capacity cores
  - Type: long
  
**Aggregation**

- Time field : @timestamp
- Inverval : auto
- Data Formatter : Custom          
- Format string : 0.0a
- Template : {{value}} nanocores
- Chart type : Line

```bash
## Average ##
cores = Average(kubernetes.node.cpu.capacity.cores)

## Bucket Script ##
expression = params.cores * 1000000000

GroupBy-Terms : kubernetes.node.name
Top : 10000
OrderBy : cores
Desc
``` 


### Memory-usage-by-node

- kubernetes.container.memory.usage.bytes
  - Total memory usage
  - type: long
  - format: bytes
  
**Aggregation**

- Time field : @timestamp
- Inverval : auto
- Data Formatter : Bytes          
- Template : {{value}}
- Chart type : Line


```bash
## Sum ##
a = Sum(kubernetes.container.memory.usage.bytes)

## CumulativeSum ##
b = CumulativeSum(a)

## Derivative ##
c = Derivative(b)/10s


GroupBy-Terms : kubernetes.node.name
Top : 10000
OrderBy : a
Desc
``` 

**Node Capacity**

- kubernetes.node.memory.capacity.bytes
  - Node memory capacity in bytes
  - type: long
  - format: bytes

**Aggregation**

```bash
## Sum ##
a = Sum(kubernetes.node.memory.capacity.bytes)

## CumulativeSum ##
b = CumulativeSum(a)

## Derivative ##
c = Derivative(b)/10s


GroupBy-Terms : kubernetes.node.name
Top : 10000
OrderBy : a
Desc
```

### Network-out-by-node

- kubernetes.pod.network.tx.bytes
  - Transmitted bytes
  - type: long
  - format: bytes

- kubernetes.node.name
  - Kubernetes node name
  - type: keyword
  
**Aggregation**

- Time field : @timestamp
- Inverval : auto
- Data Formatter : Bytes          
- Template : {{value}}
- Chart type : Line

```bash
## Max ##
a = Max(kubernetes.pod.network.tx.bytes)

## Derivative ##
b = Derivative(a)

## Positive Only ##
c = PositiveOnly(b)

GroupBy-Terms - kubernetes.node.name
Top - 10000
OrderBy : a
Desc
```



### Network-in-by-node

 - kubernetes.pod.network.rx.bytes
   - Received bytes
   - type: long   
   - format: bytes

**Aggregation**

- Time field : @timestamp
- Inverval : auto
- Data Formatter : Bytes          
- Template : {{value}}
- Chart type : Line

```bash
## Max ##
a = Max(kubernetes.pod.network.rx.bytes)

## Derivative ##
b = Derivative(a)

## Positive Only ##
c = PositiveOnly(b)

GroupBy-Terms - kubernetes.node.name
Top - 100000
OrderBy : a
Desc
```

#Top-CPU-intensive-pods

- kubernetes.container.cpu.usage.core.ns
  - Container CPU Core usage nanoseconds
  - type: long

**Aggregation**

- Data timerange mode : Last value
- Time field : @timestamp
- Inverval : auto
- Data Formatter : Custom    
- Format string : 0.0a      
- Template : {{value}} ns

```bash
## Max ##
a = Max(kubernetes.container.cpu.usage.core.ns)

## Derivative ##
b = Derivative(a)/1s

## Positive Only ##
c = PositiveOnly(b)

GroupBy-Terms - kubernetes.pod.name
Top - 10
OrderBy : a
Desc
```


### Top-memory-intensive-pods

- kubernetes.container.memory.usage.bytes
  - Total memory usage  
  - type: long  
  - format: bytes


**Aggregation**

- Data timerange mode : Last value
- Time field : @timestamp
- Inverval : auto
- Data Formatter : Bytes    

```bash
## Sum ##
a = Sum(kubernetes.container.memory.usage.bytes)

## Cumulative Sum ##
b = CumulativeSum(a)

## Derivative ##
c = Derivative(b)/10s

GroupBy-Terms - kubernetes.pod.name
Top - 10
OrderBy : a
Desc
```

