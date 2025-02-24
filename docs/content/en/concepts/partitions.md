---
description: >
  How to create shard partitions
---

# Partition API

`Partitions` and `PartitionSets` build an API that allows service providers and cluster administrators to create shard Partitions. These partitions can then be leveraged:
- for the deployment of the controllers of the service providers and other components. These APIs provide the information on shard topology required for the scheduling.
- for grouping APIExport endpoints of multiple shards in common buckets, that is `Partitions`. These buckets can the be used to achieve geo proximity: controllers should communicate with API servers of shards in the same region. They can also be leveraged for scalability and load distribution, that is to adapt the number of deployment/ processes to the load pattern specific to the controllers of a service provider.

## Partitions

The partitioning of `Shards` is done through labels and [label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors).

Here is an example of a `Partition`

```yaml
kind: Partition
apiVersion: topology.kcp.io/v1alpha1
metadata:
    name: cloud-region-gcp-europe-xdfgs
    ownerReferences:
    ...
spec:
    selector:
     matchLabels:
       cloud: gcp
       region: europe
     matchExpressions:
     - key: country
       operator: NotIn
       values:
       - LI
  ```

`Partitions` can be referenced in [`APIExportEndpointSlices`](./quickstart-tenancy-and-apis.md)

## PartitionSets

`PartitionSets` is  an API for convenience. `PartitionSet` can be used to get `Partitions` automatically created based on dimensions that match the shard label keys. The `Partitions` are created in the same workspace as the `PartitionSet`. They can then be copied to the desired workspace for consumption, for instance, by an `APIExportEndpointSlice`.

Here is an example of a `PartitionSet`

```yaml
kind: PartitionSet
apiVersion: topology.kcp.io/v1alpha1
metadata:
    name: cloud-region
spec:
   dimensions:
   - region
   - cloud
   selectors:
     matchExpressions:
     - key: region
       operator: NotIn
       values:
       - Antarctica
       - Greenland
     - key: country
       operator: NotIn
       values:
       - NK
status:
   count: 10
...
```

It is to note that a `Partition` is created only if it matches at least one shard. With the provided example if there is no shard in the cloud provider `aliyun` in the region `europe` no `Partition` will be created for it.

An example of a `Partition` generated by this `PartitionSet` can be found above. The `dimensions` are translated into `matchLabels` with values specific to each `Partition`. An owner reference of the `Partition` will be set to the `PartitionSet`.