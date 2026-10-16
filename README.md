# k8s-scheduling-patterns

> **Reference configuration.** Rendered with `kubectl kustomize` only. It was **not applied to a cluster**, so none of the placement behaviour below was observed.

Where pods land and who gets evicted when the cluster is full: spread, affinity, taints and tolerations, priority, and a disruption budget.

## The tools, in one table

| Need | Tool | Where |
|---|---|---|
| Survive losing a node or zone | `topologySpreadConstraints` | `deployment-spread.yaml` |
| Stay off certain nodes | node `taint` + pod `toleration` | `deployment-batch.yaml` |
| Go only to certain nodes | `nodeSelector` / `nodeAffinity` | both |
| Do not share a node with X | `podAntiAffinity` | `deployment-spread.yaml` |
| Who loses when the cluster is full | `PriorityClass` | `priorityclasses.yaml` |
| Do not lose too many at once during drains | `PodDisruptionBudget` | `pdb.yaml` |

## Decisions and traps

- **Taint, toleration and selector are three different things.** A taint repels, a toleration only *permits*, a nodeSelector *attracts*. Reserving nodes needs the taint and the selector.
- **Zone spread is hard (`DoNotSchedule`), node spread is soft (`ScheduleAnyway`).** A hard node constraint can leave pods Pending when nodes are few.
- **`batch-low` has `preemptionPolicy: Never`**, so batch work waits for room rather than evicting anything.
- **Preemption is a last resort**, not a capacity plan: the cluster should have headroom.
- **PDB `minAvailable: 3` with 4 replicas** lets a drain take one pod at a time. `minAvailable` equal to `replicas` would block drains forever.
- Spread constraints apply at scheduling time only; they do not rebalance running pods (a descheduler does that).

## Checking it (needs a cluster)

`kubectl get pods -n sched-demo -o wide` to see nodes; `kubectl describe pod` and read the `Events` for `FailedScheduling` reasons.

## Not checked

Everything at runtime, including whether the zone label exists on the nodes (kind clusters have none by default, so the hard zone constraint could leave pods Pending there).

MIT licensed.
