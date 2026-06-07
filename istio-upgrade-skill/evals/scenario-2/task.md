# Multi-Cluster Istio Upgrade Plan

## Problem/Feature Description

A large e-commerce platform runs Istio in a multi-primary, multi-network topology across three Kubernetes clusters: `us-east`, `us-west`, and `eu-central`. Each cluster runs its own istiod control plane and they communicate via East-West gateways. The platform has real-time inventory synchronization and checkout services that span cluster boundaries, so cross-cluster traffic cannot be interrupted for more than a few seconds during any maintenance window.

The current version is Istio 1.21 and the team needs to upgrade to Istio 1.22. The operations team has provided a snapshot of the current cluster state in `inputs/cluster_state.txt`. There is a known concern: the `eu-central` cluster's East-West gateway was last upgraded several months ago and may be running an older image.

Your task is to produce a detailed, actionable upgrade execution plan that the ops team can follow step by step during the maintenance window. The plan should account for the multi-cluster topology and ensure cross-cluster traffic remains available throughout.

## Output Specification

Produce a file called `upgrade_plan.md` containing:

- An assessment of the current multi-cluster state including any risks identified from the provided cluster snapshot
- A structured finding block (WHAT / WHEN / IMPACT / SEVERITY / FIX) for each risk, classified as Verified, Probable, Possible, or Unknown
- A numbered step-by-step upgrade execution plan for all three clusters
- A validation checklist to run after each major phase
- A rollback procedure
