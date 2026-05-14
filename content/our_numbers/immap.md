---
title: Processs
layout: default
---

# Infrastructure Metrics Masked Aggregation Protocol (IMMAP)

> “Aggregated infrastructure metrics collected through a masked sequential aggregation protocol ensuring participant confidentiality.”

## Purpose

The Infrastructure Metrics Masked Aggregation Protocol (IMMAP) allows multiple companies to contribute infrastructure-scale metrics into a shared aggregate without disclosing individual values to the poll organizer or to other participants.

The protocol is intentionally lightweight, operationally simple, and suitable for industry user groups, panels, consortiums, or ecosystem-wide benchmarking initiatives.

---

# Security Objectives

The protocol aims to provide:

| Property | Goal |
|---|---|
| Confidentiality | Individual company metrics remain private |
| Aggregation only | Only final totals are disclosed |
| Operational simplicity | No heavy cryptography required |
| Tamper visibility | Accidental modification is detectable |
| Limited trust model | Organizer cannot compute totals alone |

---

# Metrics Collected

| Metric | Unit |
|---|---|
| CPU core count | cores |
| DIMM memory | TiB |
| GPU card count | cards |
| HDD total capacity | TiB |
| SSD Read Intensive capacity | TiB |
| SSD Mixed Use capacity | TiB |
| SSD Write Intensive capacity | TiB |
| CDU count | units |
| OpenBMC host count | hosts |
| Host count | hosts |
| Global power consumption | kW |
| Datacenter room count | rooms |

All units MUST be fixed before poll initiation.

---

# Roles

## Poll Organizer

Responsible for:

- Defining the schema
- Defining participant order
- Relaying masked files between participants
- Publishing the final aggregate

The organizer MUST NOT receive individual company values.

---

## Initiating Company (Company 1)

Responsible for:

- Generating random masking values
- Creating the initial masked dataset
- Revealing the masking values only after aggregation completes

Company 1 acts as the masking authority.

---

## Participating Companies

Each participant:

- Receives the current masked aggregate
- Adds its own metrics locally
- Returns the updated file to the organizer relay

Participants never communicate directly with each other.

---

# Core Principle

For every metric:

```text
masked_total =
    random_mask
  + company_1_value
  + company_2_value
  + ...
  + company_N_value
```

At completion:

```text
final_total = masked_total - random_mask
```

Only Company 1 knows the random mask until the protocol ends.

---

# Operational Flow

## Step 1 — Schema Publication

The organizer publishes:

- Metric list
- Units
- File format
- Participant order
- Deadlines

Example:

```yaml
poll_id: premday-2026-infra
schema_version: 1
units:
  cpu_core_count: cores
  dimm_memory: TiB
  gpu_card_count: cards
  hdd_total_capacity: TiB
  ssd_read_intensive_capacity: TiB
  ssd_mixed_use_capacity: TiB
  ssd_write_intensive_capacity: TiB
  cdu_count: units
  openbmc_host_count: hosts
  host_count: hosts
  global_power_consumption: kW
  datacenter_room_count: rooms
```

---

## Step 2 — Initial Mask Generation

Company 1 generates large random positive integers for every metric.

Example:

```yaml
initial_mask:
  cpu_core_count: 19834712
  dimm_memory: 918273
  gpu_card_count: 67291
  hdd_total_capacity: 918273
  ssd_read_intensive_capacity: 72637
  ssd_mixed_use_capacity: 198273
  ssd_write_intensive_capacity: 67281
  cdu_count: 8271
  openbmc_host_count: 82731
  host_count: 62731
  global_power_consumption: 982731
  datacenter_room_count: 712
```

The mask MUST remain secret until the final aggregation stage.

---

## Step 3 — Initial Dataset Creation

Company 1 creates the first masked aggregate:

```yaml
poll_id: premday-2026-infra
schema_version: 1
step: 0

running_totals:
  cpu_core_count: 19834712
  dimm_memory: 918273
  gpu_card_count: 67291
  hdd_total_capacity: 918273
  ssd_read_intensive_capacity: 72637
  ssd_mixed_use_capacity: 198273
  ssd_write_intensive_capacity: 67281
  cdu_count: 8271
  openbmc_host_count: 82731
  host_count: 62731
  global_power_consumption: 982731
  datacenter_room_count: 712
```

Company 1 sends the file to the organizer.

---

## Step 4 — Organizer Relay Model

The organizer relays the current aggregate sequentially:

```text
Company 1
   ↓
Organizer relay
   ↓
Company 2
   ↓
Organizer relay
   ↓
Company 3
   ↓
Organizer relay
   ↓
...
```

Participants never receive files from another participant directly.

This significantly reduces inference opportunities.

---

## Step 5 — Participant Update

Each participant:

1. Receives the current masked totals
2. Adds local values
3. Updates the audit trail
4. Returns the updated file to the organizer

Example operation:

```text
new_total = previous_total + local_company_value
```

---

## Step 6 — Audit Trail

Each update appends metadata:

```yaml
audit_trail:
  - participant_alias: company-02
    step: 2
    timestamp: 2026-05-14T13:22:11Z
    sha256_after_update: "..."
```

A simple SHA-256 checksum is likely sufficient in practice for this type of industry poll.

Cryptographic signatures (GPG, SSH signatures, etc.) would improve tamper resistance and non-repudiation, but are probably unnecessary for a cooperative user-group style aggregation exercise and would significantly increase operational complexity.

Recommended minimal controls:

| Control | Recommendation |
|---|---|
| File hashing | SHA-256 |
| File format | YAML or JSON |
| Relay channel | Encrypted mail or secure upload |
| Access | One participant at a time |

---

## Step 7 — Final Aggregate Return

The last participant returns the final masked totals to the organizer.

At this point:

```text
Organizer has:
- final masked totals
- audit trail

Organizer still cannot compute real totals.
```

---

## Step 8 — Mask Reveal

Company 1 separately sends the original mask values to the organizer.

Example:

```text
final_total = final_masked_total - initial_mask
```

---

## Step 9 — Publication

The organizer publishes only aggregated totals.

No individual contribution is ever disclosed.

Example:

| Metric | Aggregate |
|---|---|
| CPU cores | 12,482,771 |
| DIMM memory | 918,221 TiB |
| GPUs | 48,172 |
| HDD capacity | 28.4 EiB |

---

# Threat Model & Limitations

## Mitigated Risks

| Risk | Mitigation |
|---|---|
| Organizer seeing raw company data | Prevented |
| Participants seeing previous participant values | Reduced |
| Accidental tampering | Detectable |

---

## Remaining Limitations

### Organizer collusion with Company 1

If Company 1 reveals masks early, confidentiality collapses.

---

### Statistical inference

Very small participant pools may still allow educated guesses.

Recommendation:

```text
Minimum recommended participant count: 8–10 organizations
```

---

### Extreme outliers

Unusually large infrastructure footprints may remain identifiable indirectly.

---

# Recommended Operational Practices

| Practice | Recommendation |
|---|---|
| Minimum participants | ≥ 10 |
| Minimum metric granularity | Large-scale only |
| No per-region splits | Avoid deanonymization |
| Use ranges publicly | Optional |
| Fixed deadlines | Yes |
| Independent audit | Optional |

---

# Optional Hardening

Future versions could use:

- Additive secret sharing
- Secure multiparty computation (MPC)
- Homomorphic encryption
- Threshold cryptography

However, IMMAP intentionally prioritizes operational simplicity over cryptographic sophistication.

---
